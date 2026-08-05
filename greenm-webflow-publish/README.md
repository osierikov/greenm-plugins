# greenm-webflow-publish

End-to-end Webflow publishing for GreenM. Covers both content types on greenm.io — **blog posts** and **case studies**: read the source, upload images, convert body/section markdown to Rich Text HTML, create the Webflow CMS item as a draft, publish, validate, and log the rollout.

## Components

Two skills:

### `webflow-publish` — blog posts (10 steps)
publish-kit + v2 markdown + images → validate images → upload assets → markdown→Rich Text → compact FAQ JSON → resolve Author/Category refs → assemble fieldData → create draft → promote to live + verify → validation suite + rollout-log + Linear comment.

Triggers: "publish blog post", "опублікувати пост", "promote draft to live", references to `publish-kit.md` / `Lucy - growth/docs/content/blog/drafts/`.

### `case-study-publish` — case studies
publish-kit + draft + `imgs/` → validate images → upload assets → draft sections→Rich Text → resolve Services Provided + Security Logos refs → assemble fieldData → create draft → **publish to staging for review** → promote to prod on explicit confirmation → validation + rollout-log + Linear comment.

Triggers: "publish case study", "опублікувати кейс", "add a new case study", references to `Lucy - growth/docs/content/case studies/`. Default target is **staging** (`greenm.webflow.io`); prod (`greenm.io`) requires explicit confirmation.

## Prerequisites
- **Webflow MCP** installed and connected; Designer open in an active tab during asset uploads.
- **Linear MCP** for closure comments.
- Access to `Lucy - growth/docs/content/` and `Lucy - growth/docs/AEO/Schema/`.

## Files
```
greenm-webflow-publish/
├── .claude-plugin/plugin.json
├── skills/
│   ├── webflow-publish/            — blog posts
│   │   ├── SKILL.md
│   │   └── references/ (aeo-rationale, cms-fields, markdown-conversion, validation-checklist, example-publish-kit)
│   └── case-study-publish/         — case studies
│       ├── SKILL.md
│       └── references/ (cms-fields, template-gotchas, example-input)
├── README.md
└── CHANGELOG.md
```

## Provenance
Blog flow battle-tested on GRO-370 (CQC frameworks, 2026-06-10). Case study flow encodes the manual publish of the five migrated case studies + the Case Studies template build (Webflow project `CLAUDE.md` §10; AEO schema GRO-513).

## Related
- [`aeo-content`](https://linear.app/greenminc/issue/GRO-399) — content-side plugin producing drafts + publish-kits
- [GRO-401](https://linear.app/greenminc/issue/GRO-401) — the Linear ticket this plugin closes
