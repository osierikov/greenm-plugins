---
name: webflow-publish
description: Publish a blog post to greenm.io Webflow CMS end-to-end — reads publish-kit, uploads images, converts body markdown to Webflow Rich Text HTML, resolves CMS references, creates the CMS item as a draft, promotes it to live, runs post-publish validation, and logs the rollout. Triggers on phrases like "publish blog post", "опублікувати пост", "ship this post to webflow", "promote draft to live", "validate published blog post", "explain AEO", "where are we with AEO", "what fields does the blog CMS have", or any reference to publish-kit.md inside Lucy - growth/docs/content/blog/drafts/. Use whenever a blog post is being shipped to Webflow.
---

# Webflow blog publishing (GreenM)

This skill encodes the end-to-end workflow for shipping a blog post to greenm.io's Webflow CMS via the Webflow MCP. It assumes the post draft and publish-kit.md already exist (produced by the aeo-content plugin or written by hand) and that the Webflow MCP is installed and connected.

The skill packages the workflow that was battle-tested on GRO-370 (CQC Sector-Specific Frameworks 2026, published 2026-06-10).

## AEO context

This skill is part of GreenM's AEO (Answer Engine Optimization) program — getting greenm.io content cited by ChatGPT, Perplexity, Gemini, Google AI Overviews, Claude. AEO differs from classic SEO: the goal is being the source the AI quotes, not just ranking in search results.

Every field in the Blog Posts CMS is designed for a specific AEO mechanism. TLDR is what AI engines quote as the cited snippet. Key Takeaways are extracted bullet-by-bullet. FAQ JSON unlocks rich results. Author Person schema strengthens E-E-A-T. See `references/aeo-rationale.md` for the full per-field rationale.

**Single source of truth for project state:** `Lucy - growth/docs/AEO/AEO-STATUS.md`. Read it when asked about current state, completed work, active workstreams, conventions, or CMS field reference. Always more up-to-date than this skill.

**Where this skill fits:** Content gets written by the `aeo-content` plugin (or by the editorial team). The output is a publish-kit + draft markdown. This skill takes those inputs and ships the post end-to-end. Together they cover the AEO pipeline from blank page to live indexable post.

If the user asks about AEO, the broader project, or anything strategic — read `AEO-STATUS.md` first, then answer.

## Inputs the workflow expects

Locate these files first. Standard location is `Lucy - growth/docs/content/blog/drafts/{post-slug}/`:

- `publish-kit.md` — Webflow CMS field mapping, FAQ JSON, image brief, external link list. Source of truth for field values.
- `blog_{slug}_{date}_v{N}.md` — final draft body (markdown with YAML frontmatter). Use the highest version number.
- `{slug}-featured.png` — Open Graph + blog hero image. Must be 1200×628 (or close — 1200×670 is acceptable, FB crops centered).
- `{slug}-thumbnail.png` — Smaller card image, 1024×536 (or close).

If files are missing or under different names, ask the user. Do not invent values.

## Site identifiers (greenm.io Webflow)

| Resource | ID |
|---|---|
| Site ID | `68ee41bcd23a92962b4775c7` |
| Blog Posts collection | `69d79f8d85db5ead564b125b` |
| Authors collection | `69d7791b1515e5facb1cb873` |
| Categories collection | `69d7821db7fcd56acd43f633` |

Verify these are still current with `data_sites_tool.list_sites` if site changes are suspected.

## Workflow

Execute the 10 steps in order. Pause at the user-confirmation gates (steps 3, 8, 9) before continuing. Do not skip steps.

### Step 1 — Locate inputs

Read `publish-kit.md` and the latest `blog_*_v*.md` file. List which files are present in the post folder. If anything from the standard inputs list is missing, ask the user.

### Step 2 — Validate image dimensions and naming

Run `file {slug}-featured.png` and `file {slug}-thumbnail.png` via bash. Confirm:

- Featured ≥ 1200px wide, aspect ratio ~1.9:1 (1200×628 ideal, 1200×670 acceptable)
- Thumbnail ≥ 1024px wide
- Filenames match `{post-slug}-featured.png` and `{post-slug}-thumbnail.png` (per `[[feedback_blog_image_naming]]`). Rename if needed.

If dimensions are off, flag to user and offer regen or proceed-anyway.

### Step 3 — Upload images to Webflow Assets

Webflow MCP `asset_tool.upload_image_by_url` requires a public HTTP URL. Local files can't be uploaded directly via MCP — there are two paths:

**Path A (preferred when user is available):**
1. Tell user to drag-drop both files into Webflow Designer → Assets panel
2. User confirms upload
3. Run `asset_tool.get_all_assets_and_folders` with `query: "assets"` (NOTE: this requires the Webflow Designer to be open in an active browser tab — if MCP errors with "Unable to connect to Webflow Designer", ask user to open the Designer link returned in the error)
4. Find the two assets by name, capture their `id` + `url` fields
5. Set `altText` on both assets via `asset_tool.update_asset` using the alt text from publish-kit's image brief

**Path B (when user is not interactive):**
1. Skip the upload step
2. Note in the output that images need manual upload before the post can be published
3. Continue with placeholder asset IDs and let the user fix later

Default to Path A.

### Step 4 — Convert body markdown to Webflow Rich Text HTML

Use Python's `markdown` library with the `extra` + `sane_lists` extensions:

1. Strip YAML frontmatter (`---...---` block at top)
2. Strip the H1 title (CMS Name field renders the title separately)
3. Strip any "Source of truth" footer note that references OneDrive (it's a read-only-mirror reminder for Linear, not for prod)
4. Pass through `markdown.markdown(md, extensions=['extra', 'sane_lists'])`
5. Post-process external links: any `<a href="...">` where href does NOT contain `greenm.io` → add `rel="nofollow noopener"` (per GreenM blog convention). Internal greenm.io links stay bare.
6. **CRITICAL — collapse inter-tag whitespace**: `html = re.sub(r'>\s+<', '><', html)`. Webflow Rich Text API silently sanitizes body content if there are newlines/whitespace between top-level tags. Newlines inside `<ul>/<ol>` become stray text nodes, which causes Webflow to drop the entire list block — and any `<hr>`, `<a>` links inside or near it. The `create_collection_items` response will echo your input verbatim, so the bug is invisible until you re-read the item or check the live page. Always emit body as one continuous string. (Discovered on GRO-370 → cqc-incident-learning-loop publish; fix Claude Code suggested 2026-06-10.)

Verify the output:
- Section headings are `<h2>`, sub-headings `<h3>`
- Bullet lists use `<ul><li>`, numbered lists `<ol><li>`
- `<strong>` for bold-prefix bullets (e.g., "**Safe**: are people protected...")
- All external CQC/data.gov.uk/etc. links carry `rel="nofollow noopener"`
- **No newlines between tags** — `assert '>\n<' not in html and '> <' not in html` should pass. If `<` is preceded by anything other than `>` directly, Webflow will drop content. Test: `re.search(r'>\s+<', html)` should return None.

See `references/markdown-conversion.md` for the full conversion ruleset and edge cases.

### Step 5 — Compact FAQ JSON

Take the FAQPage JSON-LD from publish-kit:

1. Strip the `<script type="application/ld+json">...</script>` wrapper
2. Parse to dict, dump back with `json.dumps(faq, ensure_ascii=False, separators=(',', ':'))` (single-line, no whitespace except inside strings)
3. Verify length is reasonable (CQC post: 2902 chars for 6 Q&A)

The Webflow blog template wraps the field value in `<script type="application/ld+json">` automatically on render. Do NOT include the wrapper in the field value.

### Step 6 — Resolve Author and Category slugs to ref IDs

Authors and Categories are Reference fields on Blog Posts — they require CMS item IDs, not slug strings.

Run `data_cms_tool.list_collection_items` for the Authors collection. Find the author item whose `fieldData.slug` matches publish-kit's author (e.g., `alexey-litvin`). Capture its `id`.

Same for Categories.

Common GreenM IDs (verify if reused):

| Author | Slug | ID |
|---|---|---|
| Alexey Litvin | `alexey-litvin` | `69d7807cf636a8c38e8b1984` |
| Anton Avramenko | `anton-avramenko` | `69d78103a764a0db28b4cd5c` |
| Vitalii Samarskyi | `vitalii-samarskyi` | `69d7812eac57253e3fe687c3` |
| Elena Kirienko | `elena-kirienko` | `69d7815957fea598495aa0ea` |

| Category | Slug | ID |
|---|---|---|
| Healthcare Intelligence | `healthcare-intelligence` | `69d78320a7fdb70c22f3d304` |
| Agentic AI in Practice | `agentic-ai-in-practice` | `69d7842317dd5f8b065b1850` |
| Private AI & Compliance | `private-ai-and-compliance` | `69d78471ed48e1679fac3d74` |
| Inside GreenM | `inside-greenm` | `69d784dbfaed18b739f016c7` |
| Industry Digest | `industry-digest` | `69d78506bba46f30a7cabe66` |

If user requests a new category that doesn't exist, ask whether to create one or pick the closest existing.

### Step 7 — Assemble fieldData with validation

Build the full `fieldData` dict for `data_cms_tool.create_collection_items`. The Blog Posts collection has 19 required-or-near-required fields. See `references/cms-fields.md` for the full schema.

Required-by-CMS fields:

```python
fieldData = {
  "name": "<full title from publish-kit>",
  "slug": "<kebab-case-slug>",
  "excerpt": "<2-3 sentences, 140-180 chars>",
  "tldr": "<40-80 words, ≥300 chars, complete standalone answer>",
  "post-body": "<HTML from Step 4>",
  "main-image": {"fileId": "<asset id from Step 3>", "url": "<asset url>", "alt": "<alt text>"},
  "thumbnail-image": {"fileId": "<asset id>", "url": "<asset url>", "alt": "<alt text>"},
  "category": "<category ref id from Step 6>",
  "author": "<author ref id from Step 6>",
  "read-time": "<X min read>",
  "published-date": "<ISO with .000Z, e.g., 2026-06-10T00:00:00.000Z>",
  "featured": False,
  "seo-title": "<50-65 chars>",
  "seo-description": "<140-160 chars>",
  "faq-schema-json-2": "<compact FAQ JSON from Step 5>",
  "key-takeaways": "<3-5 lines separated by \\n, no bullet markers>",
  "last-updated": "<ISO with .000Z>",
  "published-date-iso": "<short ISO, e.g., 2026-06-10T00:00:00Z>",
  "last-updated-iso": "<short ISO, e.g., 2026-06-10T00:00:00Z>"
}
```

Run validation assertions before submitting:

- `len(excerpt) in 140..180`
- `len(tldr) >= 300 and 40 <= len(tldr.split()) <= 80`
- `len(seo_title) <= 65`
- `len(seo_description) <= 160`
- `3 <= len([l for l in key_takeaways.split("\n") if l.strip()]) <= 5`
- Both ISO date fields use `T00:00:00Z` (no real time component) — they're author-curated, distinct from Webflow's auto `lastPublished`

If publish-kit is missing TLDR/Key Takeaways/Excerpt: those should now come from aeo-content plugin per GRO-399. If still missing, draft them at publish time using the workforce-recomposition post style as voice reference, and flag the gap.

### Step 8 — Create CMS item as draft

Call `data_cms_tool.create_collection_items`:

```json
{
  "collection_id": "69d79f8d85db5ead564b125b",
  "request": {
    "isDraft": true,
    "fieldData": [<fieldData from Step 7>]
  }
}
```

Capture the returned `id` (e.g., `6a2952527e8177a06bcd0ca0`). This is the CMS item ID; you'll need it for the publish step.

**Pause here** — show the user the staging URL for review (Webflow Designer → CMS → Blog Posts → find by title, click to preview). Ask if any field needs adjustment before going live.

### Step 9 — Promote to live + publish

When user confirms:

```json
[
  {"label": "flip to live", "update_collection_items": {"collection_id": "69d79f8d85db5ead564b125b", "request": {"items": [{"id": "<item id>", "isDraft": false, "fieldData": {}}]}}},
  {"label": "publish to prod", "publish_collection_items": {"collection_id": "69d79f8d85db5ead564b125b", "request": {"itemIds": ["<item id>"]}}}
]
```

Wait ~20 seconds for CDN propagation. Then curl the live URL to verify:

```bash
curl -s -L https://greenm.io/post/{slug} | grep -oE '<title>[^<]*</title>|<meta[^>]*(robots|og:|twitter:|description|canonical)[^>]*>'
```

Expected output checklist:

- HTTP 200
- `<title>` = SEO Title from CMS (not "GreenM")
- `robots` absent (page indexable)
- `og:title`, `og:description`, `og:image` present
- `twitter:card="summary_large_image"` present
- `<link rel="canonical">` present
- Two JSON-LD `<script>` blocks: @graph (BlogPosting + Person + BreadcrumbList) and FAQPage

If anything fails — check Page Settings on Blog Posts Template page, and per-item "Exclude from search engine indexing" toggle. See `references/validation-checklist.md` for the troubleshooting tree.

### Step 10 — Validation suite + logging

Generate the 5 validation URLs and present to user with instructions:

1. **Google Rich Results Test**: `https://search.google.com/test/rich-results?url=` + URL-encoded post URL → expects Articles + Breadcrumbs + FAQ = 3 valid items
2. **schema.org validator**: `https://validator.schema.org/#url=` + URL-encoded → expects 0 errors, 0 warnings
3. **GSC URL Inspection**: user opens GSC, pastes URL, clicks Request Indexing
4. **FB Sharing Debugger**: `https://developers.facebook.com/tools/debug/?q=` + URL-encoded → user clicks "Scrape Again" to build cache
5. **LinkedIn Post Inspector**: `https://www.linkedin.com/post-inspector/inspect/` + URL-encoded → auto re-scrapes on inspection

Ask user to confirm each passes. Note non-blocking LinkedIn quirk: "No author found" — LinkedIn parses `<meta name="author">` not JSON-LD @graph Person. Not a blocker.

Then write the rollout-log row in `Lucy - growth/docs/AEO/Schema/rollout-log.md` under the "Blog posts" section. Format (extend the table):

```markdown
| `/post/{slug}` | YYYY-MM-DD | 0 err / 0 warn | ✅ 1 valid | ✅ 1 valid | ✅ 1 valid | ✅ requested YYYY-MM-DD | ✅ preview OK (notes) | ✅/☐ cache status | <Linear ticket link>. Brief notes. |
```

Finally, post a closure comment on the Linear blog ticket (usually under GRO-355 H. Content Clusters or a specific blog ticket) using `save_comment` with:

- Live URL
- Validation results table
- Body editorial cleanup notes (if any)
- Pending follow-ups

Move the Linear ticket to Done.

## Common gotchas (from GRO-370 retrospective)

1. **Blog Posts Template head bindings** — When publishing the first post on a fresh template, verify Page Settings → SEO Settings → Title Tag is bound to `{Item SEO Title}` (purple chip, not static string). Same for Meta Description, OG fields, OG image (→ Main Image). If unbinded, the live `<title>` will be the static template title (e.g., "GreenM") and OG meta will be missing. Fix in Webflow Designer Page Settings before publishing live.

2. **Per-item noindex toggle (legacy)** — Existing CMS items from dev period may have "Exclude this item from search engine indexing" checked. MCP-created items default to indexable. If a post is live but shows `noindex` in HTML, open the item in Webflow CMS UI and uncheck the toggle. (Issue GRO-398 captured this.)

3. **Global canonical URL** — Site Settings → SEO → Global canonical tag URL must be set to `https://greenm.io` (no trailing slash, no www). Without this, no page emits `<link rel="canonical">`. Site-wide setting, one-time fix. (Issue GRO-403.)

4. **Image dimensions** — AI-generated images often come back at non-OG ratios (824×460 instead of 1200×628). Always validate at Step 2 and ask for regen if off. Don't upscale via Pillow — AI images degrade poorly.

5. **Webflow Designer must be active** — `asset_tool` MCP requires the Webflow Designer to be open in an active browser tab. If the tool errors with "Unable to connect to Webflow Designer", share the Designer link from the error and ask user to open it foreground.

6. **Key Takeaways line splitting** — Field is PlainText, template's JS splits on `\n` to render bullets. Submit as one string with `\n` between takeaways (no `\r\n`, no `<br>`, no bullet prefixes). JS trims each line and filters empties.

7. **Article schema is auto-emitted from CMS** — Do NOT paste a `<script type="application/ld+json">` for BlogPosting + Person + BreadcrumbList into Custom Code or the JSON-LD Schema field. The Blog Posts Template already emits a full `@graph` from CMS fields. Adding manually causes duplicates.

8. **FAQ JSON goes in `faq-schema-json-2` field, NOT custom code** — Webflow auto-wraps in `<script type="application/ld+json">` on render. Submitting the wrapper causes nested-script-tag mess.

9. **Source-of-truth footer** — Draft files often have a final paragraph like "Source of truth lives in OneDrive — this is a read-only mirror for in-Linear review." Strip this before publishing — it's an internal author note, not blog body content.

10. **Webflow Rich Text silent sanitization — THE BIG ONE.** Webflow's Rich Text API will silently drop `<ul>`, `<ol>`, `<hr>`, and any `<a>` links inside them if the input HTML has newlines or whitespace between top-level tags. The `create_collection_items` / `update_collection_items` response echoes your input verbatim, masking the bug. Detection: after every CMS write, re-read the item via `list_collection_items` and grep the stored `post-body` for `<a `, `<ul>`, `<ol>`, `<hr` — if any are missing that were in your input, you got sanitized. Fix: collapse all inter-tag whitespace via `re.sub(r'>\s+<', '><', html)` before submit. The known-good format matches what live posts store — one continuous string. External links also need `rel="nofollow noopener"` BEFORE `href` in attribute order (Webflow normalizes to this form).

## Related skills and references

- `[[reference_aeo_content_plugin]]` — content-side plugin that produces drafts + publish-kit
- `[[feedback_blog_image_naming]]` — slug-prefixed image naming convention
- `[[reference_schema_folder]]` — schema artifacts location
- GRO-370 — first end-to-end publish (CQC post) — battle test that produced this skill
- GRO-398 — Blog Posts Template head fix
- GRO-399 — aeo-content plugin: emit TLDR/Key Takeaways/Excerpt
- GRO-402 — /blog index page SEO/OG fix
- GRO-403 — Global canonical URL site-wide

## Detailed references

- `references/aeo-rationale.md` — why each CMS field matters for AEO (AI engine extraction patterns, E-E-A-T mechanics, field-to-mechanism crosswalk)
- `references/cms-fields.md` — full Webflow CMS Blog Posts collection field schema with help text
- `references/markdown-conversion.md` — markdown → Webflow Rich Text HTML conversion rules + edge cases
- `references/validation-checklist.md` — pre-publish + post-publish validation, troubleshooting tree
- `references/example-publish-kit.md` — annotated template from CQC post
