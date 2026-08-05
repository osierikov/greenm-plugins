# Example case-study inputs (OME Health)

Real structure of a case study in `Lucy - growth/docs/content/case studies/final/{Case Name}/` — a **format example, not gospel** (may contain inaccuracies; live schema wins). Two docs + `imgs/`:

```
final/OME Health/
  ome-health-publish-kit.md        # CMS/SEO/schema layer (field values + paste-ready JSON-LD)
  ome-health-case-study-draft.md   # body (labelled sections)
  imgs/
    Hero-Image.png                 # -> hero-image   (canonical: hero.*)
    OG-Image.png                   # -> og-image     (canonical: og.*)
    Ome Health logo.png            # -> client-logo  (canonical: client-logo.*)
```

Images are mapped to CMS fields **by filename** (see the image naming convention in SKILL.md). Optional per-section images use `challenge.*`, `solution.*`, `custom-1.*`, `custom-2.*`. The skill prints a `file -> field -> alt` table for confirmation before uploading; unmatched or missing-required files stop the run.

## publish-kit.md → CMS fields
The kit is **blog-shaped**; only some fields map to the Case Studies collection.

| publish-kit field | Case Studies field |
|---|---|
| Title | `hero-title` (+ `name`) |
| Slug | `slug` |
| SEO Title | `meta-title` |
| SEO / Meta Description | `meta-description` |
| Excerpt | `card-summary` |
| Category / Solution filter | `solution-category` (option ID) |
| Published Date | `published-date-iso` |
| Last Reviewed Date | `last-updated-iso` |
| Featured image / OG | `og-image` |
| TLDR, Key Takeaways, Author reference, Content cluster | **no CMS field — ignore** |
| Schema JSON-LD (Article + Quotation) | cross-check only; template head emits the @graph from CMS bindings |

## draft.md sections → CMS fields (all Rich Text unless noted)
| Draft section | Field(s) |
|---|---|
| Client (paragraph) | `client-description` |
| Client quote (`> "…" — Name, Role`) | `quote-text` / `quote-author` / `quote-role` |
| What we did | `what-we-did-intro` + `what-we-did-tagline` (the one-line summary) |
| The Challenge | `challenge` |
| How It Was | `how-it-was` |
| The Solution | `solution` |
| Key Capabilities | `capabilities` (+ `capabilities-heading`) |
| Delivery (intro + phases) | `delivery-intro-2` + `delivery` / `delivery-phase-2-2` / `delivery-phase-3-2` / `delivery-phase-4?` |
| Outcomes (with quote) | `outcomes` (quote as trailing `<blockquote>`) |
| Bespoke sections (Coach Escalation Flow, Running Cost Under Control) | `custom-section-1-*` / `custom-section-2-*` |
| Security by Design | `security` |
| Technology (prose grouped by category) | `tech-stack` — extract tool names into a flat `<ul>`; core tech only, no SaaS |
| Services Provided | `services-provided` (List of Services item IDs) |

Client name/logo/link, stats, and dates come from the draft's Client block + the kit. Missing images or fields → ask; don't invent.
