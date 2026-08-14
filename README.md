# greenm-plugins

Claude Code / Cowork plugin marketplace for GreenM.

## Plugins

- **aeo-content** — 8-stage AEO content workflow, ideation to publish-kit. Creates the drafts folder + Linear ticket pair on Stage 2.5, produces channel-appropriate publish-kit at Stage 7, includes the Featured/OG vs in-post image split with mandatory concept rationale.
- **greenm-webflow-publish** — end-to-end Webflow publishing (blog posts + case studies), Krea image generation, Data API asset upload, staging→prod, validation. Table embeds and body-image full-width Rich Text wrapping (v0.7.0+).

## Full pipeline

`aeo-content` produces the publish-kit; `greenm-webflow-publish` ships it. Install both.

## Install

1. In Cowork / Claude Code → **Add marketplace** → enter `osierikov/greenm-plugins` (this repo).
2. Install both plugins from the marketplace list.

## Update

Push a new commit that bumps the plugin's `version` in `.claude-plugin/plugin.json`. In Cowork click **Sync** on the marketplace (or run `/plugin update` in Claude Code). The new version is pulled automatically.

## Layout

```
.claude-plugin/marketplace.json     # marketplace definition (both plugins)
aeo-content/                        # content-authoring plugin
  .claude-plugin/plugin.json
  skills/aeo-content/…
  README.md  CHANGELOG.md
greenm-webflow-publish/             # publishing plugin
  .claude-plugin/plugin.json
  skills/webflow-publish/…
  skills/case-study-publish/…
  README.md  CHANGELOG.md
```
