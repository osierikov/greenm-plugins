# Markdown → Webflow Rich Text HTML conversion

Webflow Rich Text accepts a subset of HTML. Use Python's `markdown` library with `extra` + `sane_lists` extensions, then post-process external links.

## Conversion script (canonical)

```python
import re
import markdown

md = open('blog_{slug}_{date}_v{N}.md').read()

# 1. Strip YAML frontmatter
md = re.sub(r'^---\n.*?\n---\n+', '', md, flags=re.DOTALL)

# 2. Strip H1 (CMS Name field renders the title separately)
md = re.sub(r'^# .*?\n+', '', md, count=1, flags=re.MULTILINE)

# 3. Strip read-only-mirror footer ("Source of truth lives in OneDrive...")
md = re.sub(r'\n---\n\n\*Source of truth lives in OneDrive.*?$', '', md, flags=re.DOTALL)

# 4. Convert to HTML
html = markdown.markdown(md, extensions=['extra', 'sane_lists'])

# 5. Add rel="nofollow noopener" to external links (BEFORE href in attribute order)
def add_rel(m):
    full_tag = m.group(0)
    href = m.group(1)
    if 'greenm.io' in href:
        return full_tag  # internal — leave bare
    if 'rel=' in full_tag:
        return full_tag  # already has rel
    return full_tag.replace('<a href=', '<a rel="nofollow noopener" href=', 1)

html = re.sub(r'<a href="([^"]+)"[^>]*>', add_rel, html)

# 6. CRITICAL — collapse inter-tag whitespace.
# Webflow Rich Text API silently sanitizes the body if there are newlines or
# whitespace between top-level tags. Newlines inside <ul>/<ol> become stray
# text nodes, causing Webflow to drop the entire list — and any <hr>, <a> links
# inside or near it. The API echo response masks this bug. Fix: emit body as
# one continuous string matching how Webflow stores it internally.
html = re.sub(r'>\s+<', '><', html)

# Assert no inter-tag whitespace remains
assert re.search(r'>\s+<', html) is None, "Found inter-tag whitespace — Webflow will sanitize"
```

## Element mappings

| Markdown | HTML output | Webflow rendering |
|---|---|---|
| `## H2` | `<h2>` | Section heading |
| `### H3` | `<h3>` | Sub-heading |
| `**bold**` | `<strong>` | Bold text |
| `*italic*` | `<em>` | Italic |
| `[link](url)` | `<a href="url">` | External: `rel="nofollow noopener"` added. Internal greenm.io: bare. |
| `- item` | `<ul><li>` | Bulleted list |
| `1. item` | `<ol><li>` | Numbered list |
| `**Lead-in**: rest` | `<li><strong>Lead-in</strong>: rest</li>` | Bold-prefix bullets (5 Key Questions pattern) |
| `---` | `<hr/>` | Horizontal rule. See "Editorial cleanup" below. |
| `> quote` | `<blockquote>` | Blockquote (rarely used in GreenM blog) |
| ``` `code` ``` | `<code>` | Inline code |

## Editorial cleanup

Webflow rendering quirks discovered on GRO-370:

1. **`<hr/>` (horizontal rule) renders as a visible thin line.** v2 markdown commonly uses `---` to separate FAQ from closing reflection and body from Source note footer. Editorial convention:
   - Keep `<hr/>` before "Source note:" footer (separates main body from attribution)
   - Remove `<hr/>` between FAQ section and closing reflection (both are body content, divider feels redundant)
   - User can delete in Webflow Rich Text editor after publish — easier than scripting

2. **Trailing `<p>‍</p>` (zero-width joiner U+200D)** — Webflow editor sometimes adds this. Safe to remove. Doesn't render.

3. **Curly quotes** — markdown library preserves them as-is. Good — they look better than straight quotes.

4. **Em dashes** — preserved as U+2014. Per GreenM voice guide (Tier 2 self-review), v2 drafts SHOULD have zero em dashes in body. If em dashes are present, flag to user — content team may want to convert to colons or hyphen-space-hyphen.

## Webflow Rich Text sanitization (the most important pitfall)

The single most expensive bug to debug in this pipeline. Webflow's Rich Text API:

1. **Echoes input verbatim in API responses** — both `create_collection_items` and `update_collection_items` return your submitted `post-body` exactly as you sent it. This makes the bug invisible to callers who only check the response.

2. **Sanitizes content silently on store** — anything between top-level tags that isn't another tag (i.e., whitespace, newlines, stray text) breaks the parser. The parser drops:
   - The containing `<ul>`/`<ol>` block (including all items inside)
   - Any `<hr>` adjacent to a malformed list
   - Any `<a>` links inside or near the dropped block
   - `<table>` blocks if newlines are between `<thead>`, `<tbody>`, `<tr>`, etc.

3. **Only reveals the bug** when you either:
   - Re-read the item via `list_collection_items` (stored `post-body` differs from submitted)
   - View the live or staging URL (lists and links are missing visually)
   - Use `find_collection_item` and inspect `fieldData['post-body']`

### Detection

After every CMS write, re-read and compare. A simple grep check:

```python
# After create or update, fetch the item
result = list_collection_items(slug=slug)
stored_body = result['items'][0]['fieldData']['post-body']

# Count critical elements
for tag in ['<a ', '<ul>', '<ol>', '<hr']:
    assert stored_body.count(tag) == submitted_body.count(tag), \
        f"{tag} count mismatch — Webflow sanitized. Submit body with no inter-tag whitespace."
```

### Fix

Always run this single line after markdown → HTML conversion:

```python
html = re.sub(r'>\s+<', '><', html)
```

This collapses `>\n  <` and `> <` and similar to `><`. Output matches the format Webflow stores in known-good live posts.

### Attribute order for external links

Webflow normalizes external link attributes to `rel` before `href`:

```html
<a rel="nofollow noopener" href="https://...">anchor</a>
```

If you submit `<a href="..." rel="nofollow noopener">`, Webflow stores `<a rel="nofollow noopener" href="...">`. Not a critical issue, but if you compare stored vs submitted bodies, this is one source of harmless diff.

## Edge cases

### Inline HTML in markdown

The `markdown` library passes through HTML inline. If the draft has `<sup>` or `<sub>` or other inline tags, they survive intact. Verify Webflow Rich Text accepts them (most common inline tags are supported).

### Tables

Markdown tables (`| col | col |`) require the `extra` extension (already enabled). They convert to `<table><thead><tr><th>...</th></tr></thead><tbody>...</tbody></table>`.

**Webflow Rich Text does NOT support raw `<table>` tags** — earlier notes claiming otherwise were wrong; the Rich Text whitelist strips `<table>`, `<thead>`, `<tbody>`, `<tr>`, `<th>`, `<td>` on submit, leaving cell contents as flat concatenated text on the live page. Discovered on the `healthcare-genai-from-pilot-to-production` post 2026-08 — first publish showed cell values as one continuous line of text.

**Fix (built into Step 4 of the workflow):** wrap every converted `<table>...</table>` in a Rich Text embed block with inline CSS. See `references/embed-blocks.md` §1 for the exact template, the `transform_tables()` helper function, and the verification token to grep for in the post-write sanitization check.

Do NOT rely on the default `markdown.markdown()` table output — always run it through `transform_tables()` before the whitespace-collapse step.

### Images inside body

If the draft references images via `![alt](url)`, they convert to `<img alt="alt" src="url"/>`. For Webflow CMS Rich Text, images need to be uploaded to Webflow Assets first and the URL must be the CDN URL. If body has inline images, upload them to Webflow Assets via the same flow as featured/thumbnail (Step 3 of main workflow) and replace src URLs before submitting.

**Body `<img>` tags must be wrapped as Rich Text embeds for full-width display.** Rich Text renders bare `<img>` at natural pixel dimensions — an in-post diagram or schema then appears as a small island with wide side margins (fine for photos, ugly for anything technical). Run `transform_body_images()` from `references/embed-blocks.md` §2 during Step 4.7 to wrap every body `<img>` in `<div class="w-embed"><img style="width:100%;height:auto;display:block;border-radius:8px;margin:32px 0;" .../></div>`. This does NOT touch featured/thumbnail images — those are bound to `main-image` and `thumbnail-image` CMS fields via the template.

### Code blocks

Triple-backtick code blocks become `<pre><code>...</code></pre>`. Webflow Rich Text supports this. Rarely needed in GreenM blog content.

### Footnotes

`[^1]` syntax converts to `<sup>` + `<a>` if the `footnotes` extension is enabled (not by default in our pipeline). If draft uses footnotes, add `'footnotes'` to extensions list.

## Verification

After conversion, spot-check these specific patterns in the output HTML:

```python
import re
# All external links have rel
ext_links = re.findall(r'<a rel="nofollow noopener" href="([^"]+)"', html)
print(f"External (rel=nofollow) links: {len(ext_links)}")

# No internal greenm.io links have rel=nofollow (that'd hurt our own juice)
bad_internal = re.findall(r'<a rel="nofollow[^"]*" href="(https?://greenm\.io[^"]*)"', html)
assert not bad_internal, f"Internal links have rel=nofollow: {bad_internal}"

# All h2/h3 present
h_counts = {h: len(re.findall(f'<{h}>', html)) for h in ['h2', 'h3']}
print(f"Heading counts: {h_counts}")
```
