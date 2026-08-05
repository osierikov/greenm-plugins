# greenm-plugins

Claude Code / Cowork plugin marketplace for GreenM.

## Plugins
- **greenm-webflow-publish** — end-to-end Webflow publishing (blog posts + case studies).

## Install
1. In Cowork/Claude Code → **Add marketplace** → enter `OWNER/greenm-plugins` (this repo).
2. Install **greenm-webflow-publish** from the marketplace.

## Update
Push a new commit that bumps `greenm-webflow-publish/.claude-plugin/plugin.json` `version`, then
in Cowork click **Sync** on the marketplace (or run `/plugin update` in Claude Code). The new version is pulled automatically.

Layout:
```
.claude-plugin/marketplace.json     # marketplace definition
greenm-webflow-publish/             # the plugin
  .claude-plugin/plugin.json        # bump version here to ship an update
  skills/…
  README.md  CHANGELOG.md
```
