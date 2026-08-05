# Case study images with Krea

The skill generates missing case-study images with the Krea MCP, and uses Krea to bridge local files to a public URL for upload. **Never generate** the client logo or compliance/security badges — real brand assets; if missing, stop and ask the user.

## What to generate
Only when missing from `imgs/`: Hero (`hero.*`), OG (`og.*`), and section illustrations (`challenge.*` / `solution.*` / `custom-1.*` / `custom-2.*`). Style is driven by the publish-kit **image brief, used verbatim**; if the brief gives no visual style, fall back to the `greenm-design` system (flat, brand palette, no gradients).

## Model + params
- `list_models` → pick a current text-to-image model. `get_model_schema` → its input fields (prompt, size/aspect, etc.). Do not hardcode a model ID; they change.
- Optional: `get_prompting_guide` for phrasing.
- **Hero and OG come from ONE master image** — same picture, two sizes. Generate a single large master (~2000×1500) whose subject is centered with generous margins on all sides, so both crops keep the key content. Section images are generated individually per the brief.
- Use a model that accepts arbitrary width/height for the master (e.g. `bytedance/seedream-4`); `google/imagen-4` only allows fixed presets (max landscape 1280×720).

## Generate → save
1. Generate the **master** with `generate_image` (async) — the MCP widget shows it. The prompt must state a **centered subject with margins** so it crops safely to both a square/landscape hero and a 1.9:1 OG without losing anything important.
2. **Approve gate:** show the master to the user; regenerate/adjust (style, concept, seed) on request; proceed only after approval.
3. Derive the two outputs by **center-cropping + downscaling** the master (Pillow): hero (per requested size, e.g. 1024×1024 or 1600×900) and OG (1200×630). Crop from the centre; only downscale, never upscale — so the master must be larger than both.
4. Save the crops into `imgs/hero.*` and `imgs/og.*` (keep the master too). Local files / Krea asset URL feed the upload step (SKILL Step 3).

## Upload
Upload the local `imgs/` files via `data_assets_tool.create_asset` (Data API — no Designer, no token, works in both Cowork and Claude Code); see SKILL Step 3. The Krea `get_upload_url` bridge + `asset_tool.upload_image_by_url` is only a fallback for when the Designer path is the only option.

## Never generate
- `client-logo.*` — the client's real logo.
- Compliance / security badges — from the shared Security Logos collection.

If either is missing, stop and ask the user.
