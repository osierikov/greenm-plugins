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
