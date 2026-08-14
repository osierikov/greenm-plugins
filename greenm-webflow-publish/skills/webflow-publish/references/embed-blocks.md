# Rich Text embed blocks — the escape hatch for content the Rich Text HTML whitelist rejects

Webflow's Rich Text API accepts a small subset of HTML: `h1-h6, p, ul, ol, li, strong, em, a, img, blockquote, hr, code, pre`. Anything else — `<table>`, `<video>`, custom `<div>` with styling, iframes, callouts with backgrounds — is silently stripped on store. The response echoes your input verbatim (see `validation-checklist.md` §Post-CMS-write sanitization check), so you often don't discover the loss until the live page renders as flat text.

The workaround: **Rich Text embed blocks**. A `<div class="w-embed">...</div>` wrapped around otherwise-forbidden HTML tells Webflow to preserve the block as-is and render it inline. It's the standard mechanism GreenM already uses on live posts (CTAs, styled callouts, tables).

This document is the reference for **when to emit an embed** and **how the embed must be structured** for the two cases the skill handles automatically today: styled tables, and CTA callouts (Book-a-Call blocks). Add new templates below as they become recurring patterns.

## §1 Markdown tables → styled Rich Text embed

Any markdown table in the body (a `| col | col |` header row followed by `| --- | --- |` separator, then data rows) becomes a Rich Text embed on submit. Do not let the default `markdown.markdown()` output pass through — its `<table>` gets sanitized.

### Detection

Run this on the post-markdown-conversion HTML, **before** the whitespace-collapse step (Step 4.6):

```python
import re
from bs4 import BeautifulSoup  # or lxml — either works

TABLE_PATTERN = re.compile(r'<table>.*?</table>', re.DOTALL)

def has_tables(html: str) -> bool:
    return TABLE_PATTERN.search(html) is not None

def extract_tables(html: str):
    """Return list of (start, end, table_html) for each <table>...</table> in html."""
    matches = list(TABLE_PATTERN.finditer(html))
    return [(m.start(), m.end(), m.group(0)) for m in matches]
```

### Template — copy this exact structure

Wrap every table in this embed. The `<style>` block lives inside each embed (self-contained — no dependency on site stylesheet). Duplicate style blocks across multiple embeds are fine; Webflow deduplicates in the browser and the payload cost is trivial.

```html
<div class="w-embed"><style>
.blog-table-wrap { overflow-x: auto; border: 0.5px solid rgba(31,211,175,0.12); border-radius: 8px; margin: 32px 0; }
.blog-table-wrap table { width: 100%; border-collapse: collapse; font-family: 'Sora', sans-serif; font-size: 15px; line-height: 1.6; }
.blog-table-wrap th { background: #0C1E2E; color: #F0F4F8; font-weight: 600; text-align: left; padding: 14px 20px; border-bottom: 0.5px solid rgba(31,211,175,0.12); }
.blog-table-wrap td { padding: 14px 20px; color: #7A9BB5; border-bottom: 0.5px solid rgba(255,255,255,0.06); vertical-align: top; }
.blog-table-wrap tr:last-child td { border-bottom: none; }
</style><div class="blog-table-wrap"><table><thead><tr><th>{HEADER 1}</th><th>{HEADER 2}</th></tr></thead><tbody><tr><td>{CELL 1.1}</td><td>{CELL 1.2}</td></tr><tr><td>{CELL 2.1}</td><td>{CELL 2.2}</td></tr></tbody></table></div></div>
```

**Key rules:**

- **Everything on one line inside the embed.** No newlines between `<style>`, `<div>`, `<table>`, `<thead>`, `<tbody>`, `<tr>`, `<td>` tags. Same reason as body-level whitespace collapse (`re.sub(r'>\s+<', '><', ...)`) — Webflow sanitizes stray text nodes. Emit the embed as one continuous string.
- **Preserve HTML entities** in cell content — `&amp;`, `&lt;`, `&gt;`, curly quotes as UTF-8, em/en dashes verbatim.
- **Do not add `<caption>`** — Rich Text embed doesn't render it and it complicates alignment.
- **Do not nest tables** — one-level only. Multi-column data with sub-rows should be re-modelled as separate tables or as a list.

### Conversion function

Drop this into Step 4 of the workflow, after `markdown.markdown()` but before external-link post-processing:

```python
import re

TABLE_STYLE = (
    ".blog-table-wrap { overflow-x: auto; border: 0.5px solid rgba(31,211,175,0.12); border-radius: 8px; margin: 32px 0; }"
    ".blog-table-wrap table { width: 100%; border-collapse: collapse; font-family: 'Sora', sans-serif; font-size: 15px; line-height: 1.6; }"
    ".blog-table-wrap th { background: #0C1E2E; color: #F0F4F8; font-weight: 600; text-align: left; padding: 14px 20px; border-bottom: 0.5px solid rgba(31,211,175,0.12); }"
    ".blog-table-wrap td { padding: 14px 20px; color: #7A9BB5; border-bottom: 0.5px solid rgba(255,255,255,0.06); vertical-align: top; }"
    ".blog-table-wrap tr:last-child td { border-bottom: none; }"
)

def wrap_table_as_embed(table_html: str) -> str:
    """Wrap a <table>...</table> HTML block as a Webflow Rich Text embed with GreenM styling."""
    # Collapse internal whitespace to guard against sanitization
    inner = re.sub(r'>\s+<', '><', table_html.strip())
    return (
        f'<div class="w-embed"><style>{TABLE_STYLE}</style>'
        f'<div class="blog-table-wrap">{inner}</div></div>'
    )

def transform_tables(html: str) -> tuple[str, int]:
    """Replace every <table>...</table> in html with its embed-wrapped equivalent. Return (new_html, table_count)."""
    tables = TABLE_PATTERN.findall(html)
    for original in tables:
        html = html.replace(original, wrap_table_as_embed(original), 1)
    return html, len(tables)
```

### Verification

After `create_collection_items` and re-read of the stored `post-body`, the sanitization check (`validation-checklist.md` §Post-CMS-write sanitization check) must include `<div class="w-embed"` in the count. Add it to the discrepancy check:

```python
for tag in ['<a ', '<ul>', '<ol>', '<hr', '<li>', '<div class="w-embed"']:
    submitted = submitted_body.count(tag)
    stored = stored_body.count(tag)
    if submitted != stored:
        discrepancies.append(f"{tag}: submitted={submitted}, stored={stored}")
```

If a table count mismatches, Webflow either sanitized the embed (unlikely — embeds are the whitelist) or you emitted whitespace between top-level tags inside the embed (very likely — recheck the collapse regex ran).

### Post-publish check

`curl` the live page and grep for `<table>` — if the count matches `submitted_body.count('<table>')`, embeds rendered correctly. If zero tables show up live, either the embed was dropped at store time or the site's Rich Text component isn't rendering `w-embed` blocks (very unlikely on greenm.io; already proven on the healthcare-genai-from-pilot-to-production post).

## §2 Body images — full-width proportional display

Every `<img>` that appears **in the post body** (in-post diagram, schema, chart, secondary illustration) must render at the full width of the content column while preserving its aspect ratio. The default Webflow Rich Text `<img>` rendering leaves natural pixel dimensions, which shows the image as a small island in the middle of the column with wide side margins — legible for stock photos, ugly for diagrams and schemas.

Fix: wrap every body `<img>` in a `w-embed` block with inline styling. Inline styles because Rich Text doesn't apply site CSS classes to embedded images consistently — the safest form is to specify everything on the tag itself.

**Not covered by this rule:**
- **Main image / featured image** — this is bound to `main-image` CMS field via the Blog Posts template, not embedded in `post-body`. Its dimensions and layout come from the template, not from Rich Text.
- **Thumbnail image** — same, bound to `thumbnail-image`.
- **Author photos, hero images, decorative site chrome** — all live outside the CMS `post-body` field.

Only touch `<img>` tags that end up **inside `post-body`** — those come from markdown `![alt](cdn-url)` inside the draft body.

### Template — copy this exact structure

```html
<div class="w-embed"><img src="{CDN_URL}" alt="{ALT_TEXT}" style="width:100%;height:auto;display:block;border-radius:8px;margin:32px 0;"/></div>
```

Notes on the style rules:
- `width: 100%` — the image fills the content column.
- `height: auto` — preserves aspect ratio; browser calculates from natural width/height metadata.
- `display: block` — removes the inline-image baseline gap and centers correctly inside the block flow.
- `border-radius: 8px` — matches the same rounding used on the featured image and card previews across greenm.io (visual continuity).
- `margin: 32px 0` — vertical breathing room between body paragraphs and the image. Matches the table-embed spacing (`margin: 32px 0`).

### Conversion function

Drop this into Step 4 of the workflow, after `transform_tables(html)` and before the whitespace-collapse step:

```python
import re

IMG_PATTERN = re.compile(r'<img\s+([^>]*?)/?>', re.IGNORECASE)

BODY_IMG_STYLE = "width:100%;height:auto;display:block;border-radius:8px;margin:32px 0;"

def wrap_body_image_as_embed(img_tag: str) -> str:
    """Wrap a body <img ...> in a w-embed div with full-width inline styling."""
    # Strip any existing style attribute; we replace it wholesale
    inner = re.sub(r'\s+style="[^"]*"', '', img_tag, count=1, flags=re.IGNORECASE)
    # Ensure self-closed, single-line
    if not inner.rstrip('>').endswith('/'):
        inner = inner.rstrip('>').rstrip() + '/>'
    # Inject the standard style
    inner = re.sub(r'<img\s+', f'<img style="{BODY_IMG_STYLE}" ', inner, count=1, flags=re.IGNORECASE)
    return f'<div class="w-embed">{inner}</div>'

def transform_body_images(html: str) -> tuple[str, int]:
    """Replace every top-level <img> in html with a w-embed-wrapped version. Return (new_html, image_count)."""
    matches = list(re.finditer(r'<img\s[^>]*>', html, flags=re.IGNORECASE))
    for m in reversed(matches):  # right-to-left so indices don't shift
        original = m.group(0)
        html = html[:m.start()] + wrap_body_image_as_embed(original) + html[m.end():]
    return html, len(matches)
```

**Do not run this on images already inside another embed** (e.g. if a table embed happens to contain an image, which is rare but possible). Guard by first grepping the html for `<img` occurrences outside `<div class="w-embed">` regions if you emit multiple embed types.

### Verification

Extend the post-CMS-write sanitization check to count body-image embeds:

```python
for tag in ['<a ', '<ul>', '<ol>', '<hr', '<li>', '<div class="w-embed"']:
    submitted = submitted_body.count(tag)
    stored = stored_body.count(tag)
    if submitted != stored:
        discrepancies.append(f"{tag}: submitted={submitted}, stored={stored}")

# Every <img> in body must be inside an embed (unless part of a template binding, which doesn't go through post-body)
bare_img = len([m for m in re.finditer(r'<img\s', submitted_body) if 'class="w-embed"' not in submitted_body[max(0, m.start()-30):m.start()]])
if bare_img > 0:
    discrepancies.append(f"body images outside embed: {bare_img} bare <img> — will render at natural size, not full column width")
```

### Live check

After publish, view the post at a desktop breakpoint (≥1024px). Every in-post diagram/schema should span the full content column width. If any renders small with side margins — either the embed was dropped at store time (very unlikely — check post-CMS sanitization) or a bare `<img>` slipped past Step 4.7 (re-run `transform_body_images` and re-submit).

## §3 CTA callouts — the "Book a Call" block

Not part of markdown table conversion, but the same embed mechanism. Existing greenm.io posts embed this pattern between H2 sections:

```html
<div class="w-embed"><div class="blog-cta"><p>{PROMPT SENTENCE}</p><a href="#" class="js-book-call">Book a Call <span>→</span></a></div></div>
```

The styling comes from the site's `blog-cta` class (defined once in the Webflow project's global CSS, not repeated per embed). If a draft contains a `[book a call]` marker or the phrase `→ Book a Call →`, the skill can substitute this embed. Not automatic yet — flagged for a later version. For now, insert manually if the draft's editorial pattern calls for it.

## §4 What NOT to embed

- **Simple lists, headings, paragraphs** — these are already in the Rich Text whitelist; embedding them costs render performance for no gain.
- **Featured / thumbnail images** — bound to `main-image` and `thumbnail-image` CMS fields via the Blog Posts template, not embedded in `post-body`. Do NOT touch these — they render via template bindings. (Body-embedded `<img>` tags ARE wrapped — see §2 for the full-width rule.)
- **Links** — `<a href="...">` is already whitelisted. Wrapping in `w-embed` breaks anchor styling.
- **Anything with a `<script>` tag** — Webflow silently drops embedded scripts in Rich Text (only page-level Custom Code accepts scripts). If you need interactive behavior, request it as a Webflow component, not an embed.

## §5 Adding new embed templates

When a new recurring pattern emerges (e.g. quote pull-outs, comparison cards, footnote references), add a §{N} section here with:

1. **Detection rule** — regex or markdown pattern the skill scans for
2. **Template HTML** — copy-pasteable one-liner
3. **Style block** (if any) — inline `<style>` inside the embed if the pattern needs its own CSS
4. **Verification token** — a distinctive class name to grep for in the post-write sanitization check

Do not add embeds for one-off patterns. If it appears twice, template it here.
