# greenm-plugins

Claude Code / Cowork plugin marketplace for GreenM.

## Plugins

- **greenm-webflow-publish** — end-to-end Webflow publishing (blog posts + case studies), Krea image generation, Data API asset upload, staging→prod, validation. Production plugin.
- **content-20-test** — test bundle carrying the `aeo-content` skill (workspace provisioning, image concept rationale, Track-1/Track-2 image split) plus a dev snapshot of `webflow-publish` (table→embed, body-image full-width). Use this to validate changes before they graduate into the production `greenm-webflow-publish`. Not for prod install.

## Install

1. In Cowork / Claude Code → **Add marketplace** → enter `osierikov/greenm-plugins` (this repo).
2. Install the plugin you need. For the production publishing flow — install `greenm-webflow-publish`. For end-to-end AEO content authoring plus dev-branch publishing — install `content-20-test` (uninstall `greenm-webflow-publish` first if it's already installed, so skills don't double-fire).

## Update

Push a new commit that bumps the plugin's `version` in `.claude-plugin/plugin.json`. In Cowork click **Sync** on the marketplace (or run `/plugin update` in Claude Code). The new version is pulled automatically.

## Layout

```
.claude-plugin/marketplace.json     # marketplace definition (lists both plugins)
greenm-webflow-publish/             # production publishing plugin
  .claude-plugin/plugin.json
  skills/webflow-publish/…
  skills/case-study-publish/…
  README.md  CHANGELOG.md
content-20-test/                    # test bundle (aeo-content + webflow-publish snapshot)
  .claude-plugin/plugin.json
  skills/aeo-content/…
  skills/webflow-publish/…
  README.md  CHANGELOG.md
```

## When to promote content-20-test changes to production

The bundle exists so `aeo-content` (new skill) and evolving `webflow-publish` changes can be tested against real posts without disturbing the production `greenm-webflow-publish` install. When a bundle change proves out end-to-end, either:

1. Merge the improved `webflow-publish/` into the production `greenm-webflow-publish/` folder (bumping its version), and/or
2. Promote `aeo-content` into its own top-level plugin folder in this repo (with its own marketplace entry) and remove it from the bundle.

The bundle is a staging area — it's expected to churn.
