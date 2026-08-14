# Example publish-kit.md (annotated)

This is the publish-kit format used for GreenM blog posts. The CQC Sector-Specific Frameworks 2026 post (GRO-370) used a publish-kit matching this structure.

The aeo-content plugin should produce publish-kit.md in this format. As of GRO-399, the plugin will also emit TLDR/Key Takeaways/Excerpt directly into publish-kit (currently they're drafted at publish time).

```markdown
# Publish Kit — {Post Title}

Final delivery bundle for the Webflow publication of `blog_{slug}_{date}_v{N}.md`.

---

## Webflow CMS field mapping (Blog Posts collection)

| Webflow field | Value |
|---|---|
| **Name** (H1) | {Full title — same as draft H1, max 256 chars} |
| **Slug** | `{kebab-case-slug}` |
| **SEO Title** | {50-65 chars, ends with " \| GreenM"} |
| **Meta Description** | {140-160 chars} |
| **Featured Image** | `{slug}-featured.png` *(1200×628, brief below)* |
| **Thumbnail Image** | `{slug}-thumbnail.png` *(1024×536)* |
| **Featured (Switch)** | Off by default |
| **Excerpt / Card text** | {2-3 sentences, 140-180 chars — for blog index card + meta desc fallback} |
| **TLDR** | {40-80 words, ≥300 chars, complete standalone answer — what AI engines quote} |
| **Key Takeaways** | {3-5 bullets, one per line, no markers — each standalone quotable} |
| **Body (Rich Text)** | Paste markdown body from v2 (strip frontmatter first). Bind to Rich Text wrapper class. |
| **Author** | {Author Name} *(reference field — bind to Authors collection)* |
| **Category** | {Category Name} *(reference field — bind to Categories collection)* |
| **Reading Time** | ~{N} minutes |
| **Published Date** | YYYY-MM-DD |
| **Last Updated** | YYYY-MM-DD (same as Published for new posts) |

---

## FAQPage JSON-LD

For `faq-schema-json-2` field. Submit as single-line JSON (no `<script>` wrapper). 6-8 Q&A is the sweet spot.

```json
{"@context":"https://schema.org","@type":"FAQPage","mainEntity":[
  {"@type":"Question","name":"{Question 1}?","acceptedAnswer":{"@type":"Answer","text":"{Answer 1 — 2-3 sentences, self-contained}"}},
  ...
]}
```

(Pretty-printed for review here, but submit compacted via `json.dumps(faq, separators=(',', ':'))`.)

---

## Internal + external links

Internal (greenm.io URLs): bare `<a href="...">`, no rel.

External (everything else): `rel="nofollow noopener"`, opens same tab.

| Anchor | Target |
|---|---|
| {anchor text} | {URL} |
| ... | ... |

---

## Featured image brief

- **Dimensions:** 1200×628 (Open Graph + Webflow blog hero). 1200×670 acceptable, FB crops centered.
- **Concept:** {What the image shows — structural metaphor preferred over photographic}
- **Palette:** GreenM brand colors per Visual Content Rules
- **Filename:** `{slug}-featured.png` (slug-prefixed for Webflow Assets findability)
- **Thumbnail:** Same concept, cropped to 1024×536. Filename: `{slug}-thumbnail.png`
- **Alt text:** {Descriptive, mentions visual content + context}

---

## Self-review (Stage 6) results

- Em dashes: {count} (target 0 in body)
- Tier 1 forbidden words: CLEAN / FLAGGED
- Tier 2 high-risk words: {grounded uses noted}
- British English consistency: yes / no
- Question-form headings: all H2s? all FAQ H3s?
- Citations: Tier 1 sources?
- Lead-with-answer: first 40 words state the change directly?
- Closing: open question?
- Word count: {N} (target 1500-2000)

---

## Source files

In `Lucy - growth/docs/content/blog/drafts/{slug}/`:

- `blog_{slug}_{date}_v{N}.md` — final draft
- `publish-kit.md` — this file
- `{slug}-featured.png`, `{slug}-thumbnail.png` — visual assets
```

## What's missing from older publish-kits

Pre-GRO-399 publish-kits (CQC included) didn't carry **TLDR**, **Key Takeaways**, and sometimes **Excerpt** as separate fields. Those had to be drafted at publish time, looking at the v2 body's lead-with-answer paragraph and pulling 3-5 standalone insights from the body.

After GRO-399 lands, expect publish-kit to include them — verify presence before using the publish-time fallback path.
