# content-2.0-test

Temporary Cowork plugin that bundles two skills — `aeo-content` and `webflow-publish` — into a single installable package for end-to-end validation before they merge into the existing `greenm-skills` repo on GitHub.

## Why this exists

Two workflows that used to live as separate plugins:

- **`aeo-content`** — writes GreenM content (blog, service page, case study, LinkedIn, external, Substack), creates the drafts folder + Linear ticket pair (Stage 2.5, v0.6.0), outputs the publish-kit.
- **`webflow-publish`** — takes that publish-kit, ships the post to Webflow CMS, validates it, copies drafts → final (Step 11, v0.4.0).

Their final home is inside the `greenm-skills` monorepo (`github.com/greenmorg/greenm-skills`), sitting alongside the other 8 GreenM skills. Before we push there, we want to test both skills as they'll live in prod — bundled, with cross-skill handoffs actually exercised.

This plugin is that dress rehearsal. Once it passes an end-to-end test on a real post, we:

1. Push `skills/aeo-content/` and `skills/webflow-publish/` into `greenmorg/greenm-skills` (as two new skill folders alongside the existing 8).
2. Update `greenm-skills` version + CHANGELOG.
3. Team upgrades their `greenm-skills` install; they now have all skills including these two.
4. Deprecate this test bundle + the old separate `.plugin` files.

## Install (test-only)

Claude Desktop → **Cowork** tab → **Customize** → **Plugins** → **Upload local plugin** → select `content-2.0-test.plugin`.

If you already have `aeo-content` and `greenm-webflow-publish` installed as separate plugins — **uninstall them first** to avoid triggering both plugin's copies of the skill in the same chat.

## What's inside

```
content-2.0-test/
├── .claude-plugin/plugin.json         (v0.1.0)
├── README.md, CHANGELOG.md
└── skills/
    ├── aeo-content/                   (from aeo-content v0.6.0)
    │   ├── SKILL.md
    │   ├── references/
    │   │   ├── content-types.md
    │   │   ├── healthcare.md
    │   │   ├── verification.md
    │   │   ├── webflow.md
    │   │   ├── workspace-provisioning.md
    │   │   └── voice/{alexey-litvin,greenm,default,custom}.md
    │   └── assets/pre-publish-checklist.md
    └── webflow-publish/               (from greenm-webflow-publish v0.4.0)
        ├── SKILL.md
        └── references/
            ├── aeo-rationale.md
            ├── cms-fields.md
            ├── example-publish-kit.md
            ├── markdown-conversion.md
            └── validation-checklist.md
```

## Test protocol

1. **Uninstall old plugins** — `aeo-content` and `greenm-webflow-publish` if installed as separate plugins.
2. **Install this bundle** through Customize → Plugins → Upload.
3. **Open a fresh Cowork chat.**
4. **Trigger `aeo-content`** with a real request: *"Write a blog post about X for greenm.io"* (pick a real topic you actually plan to publish).
5. Walk through **Stages 1–2** (brief).
6. **Stage 2.5** should:
   - Create `content/{channel}/drafts/{slug}/` folder
   - Write `brief.md` inside
   - Create a Linear ticket in *Content & Testimonials* (or *Social Selling & LinkedIn Content 2025-2026* for LinkedIn), assigned to you
   - Show a two-line status: `📁 folder + 🎫 URL — assigned to X`
7. Walk through **Stages 3–7** (research → outline → draft → self-review → publish-kit).
8. **Generate image** via `img-for-blog` if needed; drop into the draft folder.
9. **Trigger `webflow-publish`** — ship the post.
10. **Step 11** should copy `drafts/{slug}/` → `final/{slug}/` after publish + validation.

Report any issues before we push to `greenm-skills`.

## After successful test

I'll prepare the git push package (variant B) so you can commit both skills into `greenmorg/greenm-skills`. This bundle then becomes obsolete.

---

Questions → @Sierikov.
