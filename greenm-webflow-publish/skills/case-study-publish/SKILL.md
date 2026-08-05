---
name: case-study-publish
description: Publish a case study to greenm.io Webflow CMS end-to-end — reads the publish-kit + draft, uploads images, converts the draft's sections to Webflow Rich Text HTML, resolves Services Provided and Security Logos references, creates the CMS item as a draft, publishes to staging for review (prod on explicit confirmation), runs post-publish validation, and logs the rollout. Triggers on phrases like "publish case study", "опублікувати кейс", "ship this case study to webflow", "add a new case study", "промотувати кейс на прод", "validate published case study", "what fields does the case study CMS have", or any reference to a case-study folder inside "Lucy - growth/docs/content/case studies/". Use whenever a case study is being shipped to Webflow.
---

# Webflow case study publishing (GreenM)

End-to-end workflow for shipping a case study to greenm.io's Webflow CMS via the Webflow MCP. Sibling to the `webflow-publish` (blog) skill in this plugin — same shape, different collection and template. Assumes the publish-kit, draft, and images already exist and the Webflow MCP is installed and connected.

## Where this fits

Case Studies are a single Webflow CMS collection + one Collection Page template (`detail_case-studies`). Adding a new case study = fill the CMS fields, not lay out a page. This skill takes a written case study (publish-kit + draft + images) and ships it: draft → staging review → prod.

**Default publish target: staging.** Publish to `greenm.webflow.io` first for review. Promote to the live `greenm.io` domain only on explicit user confirmation (project rule: always ask before touching live).

## Runtime modes — Claude Code vs Cowork

Detect the runtime before Step 3 by checking for a Webflow API token in the shell: `env | grep -iE 'webflow|wf_'` (common names `WEBFLOW_API_TOKEN`, `WEBFLOW_TOKEN`).

- **Claude Code (token present in shell):** everything automatic. Image upload via `data_assets_tool.create_asset` (or the raw Data API token). Only pauses are the staging-review and prod-confirmation gates.
- **Cowork (no shell token; MCP only):** also fully automatic for images — `data_assets_tool.create_asset` uploads local files via the Data API with **no Designer needed** (Step 3). Page head code (schema/CSS) is never written by MCP — one-time template setup only.

Every step except image upload behaves identically in both runtimes.

## Inputs the workflow expects

Content moves `drafts/{Case Name}/` → `final/{Case Name}/`. Publish from the **final** folder: `Lucy - growth/docs/content/case studies/final/{Case Name}/`. Two docs + an image folder (mirrors the blog flow):

- `{slug}-publish-kit.md` — the CMS/SEO/schema layer. Source of truth for field values; ships a paste-ready JSON-LD block.
- `{slug}-case-study-draft.md` — the body. Its labelled sections map to the Rich Text / text fields (see Step 4 and `references/cms-fields.md`).
- `imgs/` — the case images, named per the **image naming convention** below. The skill maps each file to its CMS field **by filename** — it does not infer placement from image content.

> **The publish-kit is blog-shaped.** It carries fields the Case Studies collection does NOT have — TLDR, Key Takeaways, Author reference, content cluster. Only these map: Title → `hero-title` (+ `name`), SEO Title → `meta-title`, Meta / SEO Description → `meta-description`, Excerpt → `card-summary`, Category / Solution filter → `solution-category`, Published / Last Reviewed → `published-date-iso` / `last-updated-iso`, Featured/OG → `og-image`. TLDR / Key Takeaways / Author / content-cluster have no CMS home here — ignore them (AEO-tracking / blog leftovers); don't invent fields for them.

If files are missing or named differently, ask the user. Do not invent values.

> The `OME Health` folder is the reference the template was first built from — a **format example, not gospel**; it may contain inaccuracies. The live CMS schema (`get_collection_details`) is the source of truth for slugs and types.

### Image naming convention (`imgs/`)

The skill places images **by filename**, not by looking at the picture. Name files exactly as below — matching is case-insensitive and extension-agnostic; the legacy OME names in parentheses are also accepted.

| File in `imgs/` | → CMS field | Required |
|---|---|---|
| `hero.*` (or `Hero-Image.*`) | `hero-image` | yes |
| `og.*` (or `OG-Image.*`) | `og-image` | no — falls back to Hero |
| `client-logo.*` (or `{Client} logo.*`) | `client-logo` | optional in `imgs/` — else pulled from shared `Logos/Clients/{Client}` |
| `challenge.*` | `challenge-image` | no |
| `solution.*` | `solution-image` | no |
| `custom-1.*` | `custom-section-1-image` | no |
| `custom-2.*` | `custom-section-2-image` | no |

Compliance/security badges are NOT in `imgs/` — they come from the shared Security Logos collection via `security-logos-2` (Step 5). Any file that matches no row, or a missing required image (Hero, Client logo), → stop and ask. Never guess placement.

Missing **generatable** images (Hero, OG, section illustrations) are created from the publish-kit image brief via Krea in Step 2b. The **client logo comes from the shared `Logos/Clients/` library** (matched by client name) when not in `imgs/`. Client logo and compliance badges are **never** generated — real assets from the shared logo library (see `references/logos.md`). Technology is text pills (`tech-stack`), not images.

## Site identifiers (greenm.io Webflow)

| Resource | ID |
|---|---|
| Site ID | `68ee41bcd23a92962b4775c7` |
| Case Studies collection | `6a5628d1699b78cee3a4502d` |
| Case Studies template page | `6a5628d1699b78cee3a45075` (`detail_case-studies`) |
| List of Services collection (Services Provided) | `6a56326e0d6a2cd9f5f6c910` |
| Security Logos collection (Security Logos) | `6a59f8908cbfa8b087d3baf3` |

Verify with `data_sites_tool.list_sites` / `get_collection_details` if changes are suspected. The Case Studies collection is at 60/60 fields — do NOT add fields; only fill `fieldData`.

## URL note (migration in progress)

Items currently render at `/case-study/{slug}` (singular) on staging, because the `case-studies` folder is still occupied by old static pages. Post-cutover the URL becomes `/case-studies/{slug}`. Keep the item `slug` identical to the old static page so ranked URLs don't change. Confirm the current path before building validation URLs.

## Workflow

Execute in order. Pause at the confirmation gates (steps 6, 8, 9). Do not skip steps.

### Step 1 — Locate inputs and read the schema

1. Read both `{slug}-publish-kit.md` (field/SEO/schema values) and `{slug}-case-study-draft.md` (body) from the final folder. List the files in `imgs/`.
2. Run `data_cms_tool.get_collection_details` on `6a5628d1699b78cee3a4502d` for current field slugs/types. Slugs are not always the display name (Stat 1 = `stat-1-4-value`/`stat-1-4-label`, Stat 2 label = `stat-2-label-5`, Delivery Intro = `delivery-intro-2`). Always read slugs from the schema; never guess. See `references/cms-fields.md`.

### Step 2 — Validate images

Map each file in `imgs/` to its CMS field via the image naming convention above, and `file`-check dimensions: Hero ≥1600px wide landscape; OG 1200×630 (if present); Client logo transparent. Build a mapping table — `file → CMS field → alt text` (alt from the publish-kit image brief where given; otherwise draft it and flag) — and **show it to the user for confirmation before uploading**. Stop on any unmatched file or missing required image (Hero, Client logo). Don't upscale AI images with Pillow — they degrade. Flag any **generatable** image (Hero, OG, section) that is missing — Step 2b creates it. Never flag the client logo or badges for generation.

### Step 2b — Generate missing images (Krea)

For each **generatable** image missing from `imgs/` — Hero, OG, section illustrations. **Never generate** the client logo or compliance/security badges (real brand assets; if missing, ask the user).

1. Build the prompt from the publish-kit image brief for that image (concept + alt), used **verbatim** as the visual direction. If the brief gives no visual style, fall back to the `greenm-design` system (flat, brand palette, no gradients). Optionally consult Krea `get_prompting_guide`.
2. Pick a model with `list_models` (+ `get_model_schema` for its inputs). Set size/aspect per target: Hero = wide landscape ≥1600px; OG = 1200×630; section = landscape unless the brief says otherwise.
3. **Hero + OG come from ONE master image** — generate a single large master (centered subject, margins, model that accepts arbitrary sizes e.g. `bytedance/seedream-4`), then after approval center-crop + downscale it to hero and OG (never upscale). Section images are generated individually.
4. Save results into `imgs/` under the convention filename (`hero.*`, `og.*`, `challenge.*`, …). See `references/image-generation.md`.
5. **Approve gate:** show the user every generated image before using it. Regenerate/adjust on request; proceed only after approval.

See `references/image-generation.md` for model selection, prompt-from-brief, sizes, and download/upload details.

### Step 3 — Upload images to Webflow Assets

Upload each file per the **confirmed Step 2 mapping** and write its returned `id` + `url` into the mapped image field in `fieldData`. The **client logo** is sourced from the shared `Logos/Clients/{Client} logo.*` (matched by client name) unless the case `imgs/` has a `client-logo.*`; upload it like any image.

**Primary path — `data_assets_tool.create_asset` (Data API). Works in BOTH Cowork and Claude Code, no Designer, no shell token** (validated on the ROC Clinic AI run):

1. md5-hash the local file (hex): `python3 -c "import hashlib,sys;print(hashlib.md5(open(sys.argv[1],'rb').read()).hexdigest())" imgs/hero.jpeg`
2. `data_assets_tool.create_asset` (`site_id`, `file_name`, `file_hash`) → returns `id`, `hostedUrl`, `uploadUrl`, and `uploadDetails` (S3 form fields).
3. multipart `POST` the bytes to `uploadUrl` with every `uploadDetails` value as a form field — **renaming the camelCase keys to the S3 field names** — and the `file` field **last**. `201` = success. Mapping: `xAmzAlgorithm`→`X-Amz-Algorithm`, `xAmzCredential`→`X-Amz-Credential`, `xAmzDate`→`X-Amz-Date`, `xAmzSignature`→`X-Amz-Signature`, `policy`→`Policy`, `successActionStatus`→`success_action_status`, `contentType`→`Content-Type`, `cacheControl`→`Cache-Control`; `key`/`acl`/`bucket` unchanged.
4. Use the returned `id` + `hostedUrl` in the image `fieldData`: `{"fileId": <id>, "url": <hostedUrl>, "alt": "…"}`.

**Fallbacks** (only if `create_asset` is unavailable):
- Claude Code with `$WEBFLOW_API_TOKEN` in the shell — same flow via raw `POST https://api.webflow.com/v2/sites/{siteId}/assets` (`{fileName, fileHash}` → S3 POST).
- `asset_tool.upload_image_by_url` from a **public URL** — requires the Webflow **Designer open in an active tab** (connects flakily, hence the Data API path is preferred); get a public URL for a local file via the Krea bridge (`get_upload_url` → POST file → public URL).
- Manual drag-drop of `imgs/*` into Designer → Assets, then read `id`/`url` via `data_assets_tool.list_assets`.

### Step 4 — Convert draft sections to Webflow Rich Text HTML

The variable-length body sections are Rich Text. Convert markdown → HTML (Python `markdown`, `extra` + `sane_lists`), then apply the **CRITICAL whitespace collapse** (gotcha #1). Draft-section → field mapping:

- Client paragraph → `client-description`; the client quote (`> "…" — Name, Role`) → `quote-text` / `quote-author` / `quote-role`
- What we did → `what-we-did-intro` (paragraph) + `what-we-did-tagline` (the one-line summary)
- The Challenge → `challenge` (`<p>` + optional `<ul>`)
- How It Was → `how-it-was` (`<ul>`)
- The Solution → `solution` (`<p>` + `<ul>`)
- Key Capabilities → `capabilities` (`<ul>`, renders as pills) + `capabilities-heading`
- Outcomes → `outcomes` (intro `<p>` + `<h3>`+`<p>` pairs + trailing `<blockquote>` for the outcomes quote — quote lives INSIDE this field)
- Delivery → `delivery-intro-2` (lead-in) + `delivery` / `delivery-phase-2-2` / `delivery-phase-3-2` / `delivery-phase-4` (one phase per field: `<h3>` name + `<ul>`)
- Bespoke sections (e.g. "Coach Escalation Flow", "Running Cost Under Control") → `custom-section-1-*` / `custom-section-2-*`
- Security by Design → `security` (intro `<p>` + `<h3>`+`<p>` pairs)
- Technology → `tech-stack` — flat `<ul><li>name</li></ul>` of technology names. **Core tech only** (languages, frameworks, cloud/infra, databases, AI tooling). Do NOT list SaaS/business-app integrations (e.g. Front). Simple names, consistent across case studies. (The draft's Technology section is prose grouped by category — extract the tool names into a flat list.)
- Services Provided → `services-provided` reference IDs (Step 5)

### Step 5 — Resolve reference fields

Both MultiReference (arrays of CMS item IDs, not slugs):

- `services-provided` → List of Services (`6a56326e0d6a2cd9f5f6c910`). `list_collection_items`, match by name, collect IDs.
- `security-logos-2` → Security Logos (`6a59f8908cbfa8b087d3baf3`), sourced from `Logos/commpliance/`. Select the applicable badges; items must be **non-draft**. Never generated.
Technology is **not** a reference — it is the text `tech-stack` field (pills, core tech only, no SaaS). There is no tech-logo grid (see `references/logos.md` for why).

### Step 6 — Assemble fieldData

Build `fieldData` keyed by slug. Required: `name`, `slug`, `meta-title`, `meta-description`, `card-summary`, `solution-category` (option ID), `hero-title`, `hero-subtitle`, `hero-image`, `client-name`, `client-logo`, `client-description`, `client-link-url`, `what-we-did-intro`, `what-we-did-tagline`, `challenge`, `how-it-was`, `solution`, `capabilities-heading`, `capabilities`, `outcomes`, `delivery`, `delivery-phase-2-2`, `delivery-phase-3-2`, `security`, `cta-heading`, `cta-button-label`, `cta-button-link`. Optional: stats, quote, images, tech-stack, custom sections, delivery-phase-4, impacts, ISO dates, references.

Solution Category option IDs: AI MVP `f3698b986a5562c21bc92c4fd811da00` · Private AI `52dcb143a058b34b2e72bcf35ce78f3d` · Unified data `81899337b2bf5ece02581d17b3af3a44` · Workflow integration `985781462a5d097df3c7f42889159db5` · Agentic AI `e284c0b63d55f545c4eb8113047c0585` · Scaling & optimization `d3a1e853a14e46350aa559a185d4332c`.

Image fields take `{"fileId","url","alt"}`. ISO date fields use short ISO `2026-07-10T00:00:00Z`.

**Pause** — show the assembled fields for review before writing.

### Step 7 — Create CMS item as draft

`create_collection_items` with `isDraft: true`, `fieldData: [<dict>]`. Capture the item `id`.

**Immediately re-read** via `list_collection_items` and grep every Rich Text field for `<ul>`, `<ol>`, `<a `, `<blockquote>`, `<hr` — if any present in input are missing, Webflow silently sanitized them (gotcha #1). Fix and re-write.

### Step 8 — Publish to staging (default)

1. `update_collection_items` with `isDraft: false`.
2. `data_sites_tool.publish_site` with `publishToWebflowSubdomain: true`, `customDomains: []` (Site ID `68ee41bcd23a92962b4775c7`) — pushes to `greenm.webflow.io` only, not `greenm.io`.

**Pause** — give the user the staging URL (`https://greenm.webflow.io/case-study/{slug}`) to review across breakpoints. Ask for adjustments.

### Step 9 — Promote to prod (explicit confirmation only)

Only when the user confirms:

```json
[{"label": "publish item to prod", "publish_collection_items": {"collection_id": "6a5628d1699b78cee3a4502d", "request": {"itemIds": ["<item id>"]}}}]
```

Verify the live URL (path per the migration note):

```bash
curl -s -L https://greenm.io/case-studies/{slug} | grep -oE '<title>[^<]*</title>|<meta[^>]*(robots|og:|description|canonical)[^>]*>'
```

Expected: HTTP 200; `<title>` = Meta Title; robots not `noindex`; og:title/description/image present; canonical present; JSON-LD @graph (Article + Organization + BreadcrumbList + Quotation).

### Step 10 — Validation + logging

1. Google Rich Results Test + schema.org validator (expect Article + Breadcrumb valid, 0 errors).
2. GSC URL inspection → Request Indexing.
3. FB Sharing Debugger + LinkedIn Post Inspector to refresh social cache.
4. Rollout-log row in `Lucy - growth/docs/AEO/Schema/rollout-log.md`, Case studies section.
5. Closure comment on the Linear ticket (`save_comment`) with URL + validation; move ticket to Done.

## AEO / schema

Case study JSON-LD lives in the template's **Page Settings → Custom Code → head** (not the SEO JSON-LD field), bound to CMS fields — a new item inherits schema automatically. @graph = Article (headline←Hero Title, description←Meta Description, image←OG Image, datePublished←`published-date-iso`, dateModified←`last-updated-iso`, publisher `@id #organization`, about←Client Name) + BreadcrumbList (Success Stories→case) + Quotation (Quote Text/Author/Role). No FAQPage. Snippet: `Lucy - growth/docs/AEO/Schema/case-studies/webflow-snippet.md`. The publish-kit also ships a paste-ready JSON-LD (Article + Quotation, with `[FEATURED-IMAGE]` / `[PUBLISH-DATE]` / URL placeholders) — use it to cross-check values, but the template's CMS-bound @graph is what actually renders. MCP cannot write page head code — verify/add in Designer. See GRO-513 (under GRO-347).

## Common gotchas

1. **Webflow Rich Text silent sanitization — THE BIG ONE.** Webflow drops `<ul>`, `<ol>`, `<hr>`, and `<a>` inside them if input HTML has whitespace/newlines between top-level tags. The write response echoes input verbatim, masking it. Fix: `html = re.sub(r'>\s+<', '><', html)` before submit — emit each Rich Text field as one continuous string. Detect by re-reading the item (Step 7).
2. **Empty stat cards** — stats bind CMS text directly, so empty fields get `w-dyn-bind-empty`. Template head CSS hides fully-empty cards and stretches the rest: `.cstpl-stat-val.w-dyn-bind-empty,.cstpl-stat-lab.w-dyn-bind-empty{display:none}` + `.cstpl-stat:has(.cstpl-stat-val.w-dyn-bind-empty):has(.cstpl-stat-lab.w-dyn-bind-empty){display:none}`; container `cstpl-stats` = grid `repeat(auto-fit,minmax(200px,1fr))`. A label without a value is a valid text badge — keep it.
3. **Empty delivery phases** — `cstpl-grid3` flex-wrap; cards `cstpl-rich-text-delivery-card` = `flex:1 1 0` + min-width 260px; empty phases hidden via `.cstpl-rich-text-delivery-card.w-dyn-bind-empty{display:none}`. Leave `delivery-phase-4` empty for 3-phase cases.
4. **Reference items must be non-draft** — Services / Security Logos render only when not Draft.
5. **Head CSS/schema don't apply in the Designer canvas** — only on published/preview. Check on staging.
6. **Field limit 60/60** — collection is full. Fill `fieldData` only; the legacy `outcomes-quote-text-author-role` slot is spare if a field is ever needed.
7. **Any edit flips the item to "Changes in draft"** — to see it on staging, set `isDraft:false` and publish the site (Step 8).
8. **Client Link URL is required.** The **client logo** is required too but comes from the shared `Logos/Clients/{Client} logo.*` (matched by client name) or `imgs/client-logo.*` — never generated. Compliance badges are library assets too; technology is text pills (`tech-stack`), not logos (see `references/logos.md`).
9. **Asset S3 upload field names** — `data_assets_tool.create_asset` returns `uploadDetails` in camelCase; the S3 POST needs them renamed (`xAmzSignature`→`X-Amz-Signature`, `policy`→`Policy`, `successActionStatus`→`success_action_status`, `contentType`→`Content-Type`, `cacheControl`→`Cache-Control`, etc.) and the `file` field **last**. `201` = success.

## Detailed references

- `references/cms-fields.md` — full Case Studies field schema (slug, type, required, Rich Text shape)
- `references/template-gotchas.md` — stat/delivery empty-card CSS, rich-text sanitization, publishing gotchas
- `references/example-input.md` — annotated OME Health inputs (publish-kit + draft); a format example, may contain inaccuracies
- `references/image-generation.md` — Krea image generation (what to generate, model/size, save-to-imgs, Cowork upload bridge)
- `references/logos.md` — shared logo library (clients, compliance/Security Logos) and per-type publishing rules (technology stays text pills)

## Related

- `webflow-publish` — sibling blog-publishing skill (shared Rich Text sanitization fix, image upload, validation)
- Krea MCP — image generation + `get_upload_url` bridge (see `references/image-generation.md`)
- `greenm-design` / `img-for-blog` skills — GreenM visual system and image-prompt patterns
- Webflow project `CLAUDE.md` §10 — Case Studies migration status, template classes, gotchas
- `case-studies-cms-plan.md`, `binding-map.md` — schema plan + element↔field map (Webflow project folder)
