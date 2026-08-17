# Changelog

## [0.7.1] — 2026-08-17

### Fixed
- **Stage 7b no longer stalls the image workflow by treating the concept as a proposal awaiting approval.** Two observed failure modes:
  1. `cqc-no-inspection-for-years` (v0.6.4) — skill wrote the brief, closed with "I have not generated anything yet."
  2. `compliance-gap-evidence-mapping-problem` (v0.7.0) — skill wrote the rationale, then listed "cover image, schemas, CTA blocks, publication timing" as a numbered list of decisions for the user, leaving generation postponed. Krea MCP was connected; skill did not use it until the user asked "чому немає зображень?"
- SKILL.md Stage 7b now explicitly requires the rationale AND the generation to land in the same message. Redirect is allowed after generation; stall before generation is not. Explicit forbidden phrasings: "run `img-for-blog` against this brief," "cover image approved for generation" as a to-do item, "N decisions for you: cover, schemas, ..." lists.
- If a connected image generator (Krea MCP, Nano Banana bridge, etc.) is available, skill uses it directly and saves to `img/{slug}-cover.png` / `img/{slug}-NN.png` in the same turn. If not, `img-for-blog` runs inline and delivers a ready-to-paste prompt.

## [0.7.0] — 2026-08-14

### Changed
- **Unified image naming convention.** Every image in a draft folder now uses the full post slug as prefix:
  - **Cover** — `img/{slug}-cover.png` (1200×630, one file only). The 1024×536 thumbnail is derived on upload by `webflow-publish` v0.7.1+, not stored on disk.
  - **In-post schemas** — `img/{slug}-01.png`, `img/{slug}-02.png`, … numbered by **order of appearance in the body** (top-to-bottom), not order of generation. Optional sidecar `img/{slug}-NN-embed.html` for Rich Text embed use.
- Stage 7a and Stage 7b in SKILL.md updated with the new convention as a mandatory rule. Terminology switched from "Featured / OG" to "Cover" throughout. Concept rationale sections retitled accordingly.
- `references/workspace-provisioning.md` §5 folder scaffold updated with the new file list.
- `references/content-types.md` CMS field list references `Cover Image` (with auto-thumbnail note) instead of `Featured Image`.
- `assets/pre-publish-checklist.md` now has a dedicated "Image files in `img/`" section enforcing the naming rules.

### Rationale
- Every asset in Webflow Assets (a flat list across the site) stays scannable and unambiguous.
- Draft folder is sortable in Finder — all files for one post group together.
- Regenerating a specific schema no longer requires guessing which "schema-a" it was.
- `webflow-publish` can grep `{slug}-*.png` deterministically without listing files first.

### Deprecated
- `{slug}-featured.png` + `{slug}-thumbnail.png` (old two-file cover pattern).
- Ad-hoc names in `img/`: `schema-a.png`, `decision-tree.png`, `figure1.png`, bare `cover.png` without slug prefix.
- Existing published posts keep their old-named assets — do not rename what's already live in Webflow. The convention change applies to drafts on disk and future publishes only.

## [0.6.5] — 2026-08-14

### Fixed
- **Stage 7b Featured/OG brief no longer stops at "I have not generated anything yet."** On the cqc-no-inspection-for-years test run the skill wrote a strong concept + legibility-gate rationale for the Featured image, then closed with "To produce the actual generation prompt, run the img-for-blog skill against this brief. I have not generated anything yet." — leaving the workflow suspended and no image on disk. SKILL.md Stage 7b now explicitly requires invoking `img-for-blog` immediately after the four gates pass and the user approves the concept: call the skill (or inline its process) so a ready-to-paste generation prompt lands in the same message. If a connected image generator is available (Krea MCP, Nano Banana bridge, etc.), the skill uses it to produce the file directly into `content/{channel}/drafts/{slug}/img/` with the correct name. Otherwise it hands the user the prompt and picks up center-cropping when the file returns.

## [0.6.4] — 2026-08-14

### Fixed
- Shortened `plugin.json` and marketplace.json `description` fields from 568 to ~200 characters. Cowork Directory silently omitted the plugin from the marketplace list when description exceeded ~250 chars (undocumented limit, discovered when Sync showed only `greenm-webflow-publish`). All functional content preserved elsewhere — full workflow details live in README.md and SKILL.md, not in the manifest description.

# Changelog

## 0.1.0 — 2026-08-13

Initial test bundle. Combines two skills for end-to-end pipeline validation before push to `greenmorg/greenm-skills`.

### Included skills

- **`aeo-content`** (from `aeo-content` plugin v0.6.0)
  - 8-stage AEO content workflow (Ideation → Brief → Provision → Research → Outline → Draft → Self-review → Publish-kit → Post-publish)
  - Channel-aware (blog, service page, case study, LinkedIn, external, Substack, other)
  - Voice-aware (Alexey Litvin, GreenM brand, Universal, Custom)
  - Stage 2.5 workspace provisioning — creates drafts folder + Linear ticket pair, assigns to initiator
  - TLDR/Key Takeaways/Excerpt as first-class CMS fields synced to Webflow hints
- **`webflow-publish`** (from `greenm-webflow-publish` plugin v0.4.0)
  - 11-step Webflow CMS publishing workflow (locate → validate images → upload → convert markdown → compact FAQ → resolve refs → assemble → create draft → publish → validate → copy to final)
  - Webflow Rich Text sanitization fix (collapse inter-tag whitespace)
  - Full validation suite (Rich Results Test, schema.org, GSC, FB Debugger, LinkedIn Post Inspector)
  - Copies `drafts/{slug}/` → `final/{slug}/` after successful publish

### Notes

- Plugin name `content-2.0-test` produces a claude.ai marketplace validator warning (period is not kebab-case). Cowork accepts it. Warning is safe for local testing; the destination repo (`greenm-skills`) uses proper kebab-case names.
- Skill invocation names remain `aeo-content` and `webflow-publish` — identical to what they'll be inside `greenm-skills` after push.
- Uninstall the old separate plugins (`aeo-content` and `greenm-webflow-publish`) before installing this bundle to avoid double-firing.
