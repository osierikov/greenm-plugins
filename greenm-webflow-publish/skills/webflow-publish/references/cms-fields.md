# Webflow CMS — Blog Posts collection field schema

Collection ID: `69d79f8d85db5ead564b125b`
Singular name: "Blog Post"
Site: greenm.io (`68ee41bcd23a92962b4775c7`)

Full field list as returned by `data_cms_tool.get_collection_details`. Help text quoted verbatim.

## Required-by-CMS fields

### `name` (Title)

- Type: PlainText, max 256 chars
- Use: H1 of the post page + breadcrumb last item
- No help text in CMS

### `slug`

- Type: PlainText, alphanumeric + hyphens, max 256
- Use: URL segment, becomes `/post/{slug}`
- Must be unique within collection. Renaming breaks all links referencing the old slug.

### `excerpt`

- Type: PlainText (multi-line)
- Help text: "2–3 sentences. Used in post card and as fallback for meta description. 140–180 chars."
- Use: blog index card preview + meta description fallback if SEO Description empty

### `tldr`

- Type: PlainText (multi-line), min 300 chars
- Help text: "40–80 word answer to the post's main question. Shown at top of article and used by AI engines as the cited snippet. Write a complete standalone answer, not a teaser."
- Critical for AEO — this is what ChatGPT/Perplexity/Gemini quote

### `post-body`

- Type: RichText
- Accepts HTML: h1-h6, p, ul, ol, li, strong, em, a, img, blockquote, hr, code, pre
- External links: add `rel="nofollow noopener"` per GreenM convention

### `main-image`

- Type: Image
- Format: `{"fileId": "<asset id>", "url": "<asset cdn url>", "alt": "<alt text>"}`
- Used as Open Graph image + blog hero
- **Source file:** `content/{channel}/drafts/{slug}/img/{slug}-cover.png`, 1200×630 exact (produced by `aeo-content` Stage 7b)

### `thumbnail-image`

- Type: Image, same format as main-image
- Help text: "Smaller version of main image that is used on blog post grid"
- Used on blog index card
- **Derived, not authored:** `webflow-publish` Step 3 resizes the cover PNG to 1024×536 with Pillow (`Image.LANCZOS`), uploads under the filename `{slug}-cover-thumbnail.png`, and binds the resulting asset here. No separate thumbnail file lives in the draft folder on disk.

### `category`

- Type: Reference to Categories collection (`69d7821db7fcd56acd43f633`)
- Value: just the category item's ID string, e.g., `"69d78471ed48e1679fac3d74"`

### `author`

- Type: Reference to Authors collection (`69d7791b1515e5facb1cb873`)
- Value: just the author item's ID string

### `read-time`

- Type: PlainText (single line)
- Convention: human string like `"5 min read"` or `"10 min read"`
- NOT a number

### `published-date`

- Type: DateTime
- Format: ISO with milliseconds, e.g., `2026-06-10T00:00:00.000Z`
- Author-curated, distinct from Webflow's auto-set `lastPublished`

### `seo-title`

- Type: PlainText (multi-line)
- Help text: "Separate field from Title. 50–60 characters. Bound to tag in Webflow."
- Bound to template `<title>` and OG title via "Same as SEO title tag"

### `seo-description`

- Type: PlainText (multi-line)
- Help text: "150–160 characters. Bound to meta description. Include focus keyword naturally."

### `key-takeaways`

- Type: PlainText (multi-line)
- Help text: "3–5 standalone bullets, one per line. Each bullet must make sense quoted out of context — AI engines extract individual lines."
- Format: lines separated by `\n`. NO bullet markers (`-`, `*`, `•`). Template JS splits on `\n`, trims, filters empty, and renders as `<ul><li>` automatically.

### `last-updated`

- Type: DateTime
- Help text: "Set manually when you make meaningful content changes (not for typos). Used as dateModified in schema — AI deprioritises content older than 3–6 months."
- For new post: same as `published-date`

### `published-date-iso`

- Type: PlainText (single line)
- Help text: "Full ISO 8601 with timezone, e.g., 2026-05-26T00:00:00Z. Use T00:00:00Z when exact time unknown. Required for schema.org datePublished validation."
- Format differs slightly from `published-date` — no `.000` milliseconds segment

### `last-updated-iso`

- Type: PlainText (single line)
- Help text: "Full ISO 8601 with timezone, e.g., 2026-05-29T00:00:00Z. Used as dateModified in schema. Use T00:00:00Z for date-only updates."

## Optional fields

### `featured`

- Type: Switch (boolean)
- Help text: drives homepage feature module via Conditional Visibility
- Default `false` unless explicitly editorial-promoted

### `faq-schema-json-2`

- Type: PlainText (multi-line), optional
- Help text: "FAQ microdata"
- Content: single-line FAQ JSON-LD object (without `<script>` wrapper — template auto-wraps)
- If post has no FAQ section, leave empty — FAQPage block won't render

## Auto-managed by Webflow (don't set via API)

- `lastPublished`, `lastUpdated`, `createdOn` — system timestamps
- `cmsLocaleIds` — locale assignment

## Authors collection schema

Collection ID: `69d7791b1515e5facb1cb873`

| Field | Type | Required |
|---|---|---|
| `name` | PlainText | yes |
| `slug` | PlainText | yes |
| `photo` | Image | yes |
| `role` | PlainText | yes |
| `bio-2` | PlainText | yes |
| `linkedin` | Link | yes |
| `meta-title` | PlainText | yes |
| `meta-description` | PlainText | yes |
| `credentials` | PlainText | optional |

## Categories collection schema

Collection ID: `69d7821db7fcd56acd43f633`

| Field | Type | Required |
|---|---|---|
| `name` | PlainText | yes |
| `slug` | PlainText | yes |
| `description` | PlainText | optional |
| `color` | Color | optional |
| `seo-title` | PlainText | optional |
