# aeo-content

GreenM AEO content-authoring plugin. Turns a topic into a publish-ready draft + publish-kit + provisioned Linear ticket + workspace folder, ready to hand off to `greenm-webflow-publish` for the actual Webflow ship.

## Skill

- **aeo-content** — the 8-stage workflow (ideation → brief → provision → research → outline → draft → self-review → publish-kit → post-publish AEO baseline). Channel- and voice-aware. Fires on any request to write, edit, or review greenm.io content.

## Pairs with

- **greenm-webflow-publish** (same marketplace) — takes the publish-kit produced here and ships the post to Webflow with validation.

## Install

Marketplace: `osierikov/greenm-plugins`. Install `aeo-content` and `greenm-webflow-publish` together for the full pipeline.

## What it produces

For every content invocation, at Stage 2.5:
- `content/{channel}/drafts/{slug}/brief.md` — persistent record of the brief
- Linear ticket (Content & Testimonials / Social Selling projects), assigned to the initiator, with the folder path and workflow checklist in the description

For every finished piece, at Stage 7:
- `publish-kit.md` with CMS field mapping, TLDR / Key Takeaways / Excerpt at Webflow's exact constraints, FAQ JSON-LD, image brief with concept rationale, internal links, self-review results
- `blog_{slug}_{YYYY-MM-DD}_v{N}.md` — final draft body
