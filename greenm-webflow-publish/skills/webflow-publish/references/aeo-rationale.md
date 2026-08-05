# AEO rationale — why each CMS field matters

This doc explains the *why* behind every field in the Blog Posts CMS. The publishing skill executes the mechanics; this doc explains the strategy.

## What is AEO

**AEO (Answer Engine Optimization)** is the discipline of getting your content cited by AI answer engines — ChatGPT, Perplexity, Gemini, Google AI Overviews, Claude.

AEO differs from classic SEO in three concrete ways:

| Dimension | SEO | AEO |
|---|---|---|
| **Goal** | Rank in search results so users click to your page | Be the source the AI quotes in its answer |
| **Unit of value** | A click | A citation + a sentence quoted |
| **Content shape** | Keyword-targeted page, ranked vs competitors | Structured, atomic, quotable chunks an AI can extract |
| **What AI engines actually do** | Crawl pages, build inverted index, rank pages by relevance + authority | Crawl pages, parse structure, extract specific spans, cite source per span |

Practical consequence: AI engines reward content that's **structured for extraction** — they pull specific paragraphs, bullet lines, FAQ answers, table rows. A wall of prose with no structural anchors gets less usage even if topically perfect.

This is the operating principle behind every CMS field below.

## Field-by-field rationale

### `tldr` — 40-80 words, ≥300 chars, complete standalone answer

**Why this format:** AI engines look for a quotable chunk that contains the article's main answer. 40-80 words is the sweet spot — short enough to fit in an AI's response without quoting half the article, long enough to be a complete standalone answer. Below 300 chars usually means it's a teaser not an answer.

**Why "complete standalone":** When ChatGPT cites this paragraph, the user reads only that paragraph. Not your headline, not your hero image, not the body. If the paragraph requires reading the rest of the article to make sense, the AI can't use it. Write it like a press-release lede that could stand alone on social.

**Anti-pattern:** "In this article, we'll explore..." or "Read on to learn..." — these are teasers, not answers. AI engines skip them.

**Where it ends up:** Auto-emitted in `<meta name="description">`, `og:description`, `twitter:description`, and the BlogPosting JSON-LD `description` field. So one well-written TLDR feeds 4 places.

### `key-takeaways` — 3-5 standalone bullets, one per line, no markers

**Why this format:** AI engines extract list items **individually**. A user asking "what are the main risks of X" might get just bullet #3 of your post, surrounded by competitors' bullets. Each bullet must therefore be a self-contained insight with enough specificity (numbers, names, dates, mechanisms) to be quotable out of context.

**Anti-pattern:** "Key infrastructure investments" — too vague, gets cut. "By end of 2026 the CQC Single Assessment Framework is replaced by four sector-specific frameworks" — specific, dated, quotable.

**Why no bullet markers:** The Webflow template's JavaScript renders bullets visually. Putting `-` or `•` in the field value would either show twice or break the split.

**Why 3-5:** Below 3 it's not a list, above 5 dilution starts — each bullet competes with the others for AI attention.

### `excerpt` — 140-180 chars, 2-3 sentences

**Why this format:** Used in two places — the blog index card preview AND as fallback meta description if SEO Description is empty. 140-180 chars is the meta description sweet spot (Google truncates ~155 desktop, ~120 mobile).

**Different from TLDR:** Excerpt sets up *curiosity* (drives clicks); TLDR delivers the *answer* (drives citations). They overlap topically but optimize for different metrics. Excerpt opens with the angle ("not X, Y"), TLDR opens with the substantive fact.

**Where it ends up:** Blog index card + meta description fallback. Not auto-fed to OG or schema (those come from `seo-description`).

### `seo-title` — 50-65 chars, ends with "| GreenM"

**Why this format:** Title tag is the most weighted on-page signal for traditional SEO and is what users see in search results. Google truncates at ~60 chars on desktop, ~50 on mobile. Brand suffix "| GreenM" is editorial convention — reinforces brand entity recognition.

**AEO consequence:** AI engines also use title for source attribution ("according to GreenM's article on...") — a clear title makes attribution attractive.

**Where it ends up:** `<title>` tag, `og:title`, `twitter:title`, BlogPosting JSON-LD `headline`, breadcrumb item.

### `seo-description` — 140-160 chars

**Why this format:** Meta description rules are unforgiving — Google truncates aggressively. Different audience from TLDR: this is for the *search result snippet*, where the user hasn't clicked yet and is deciding.

**Where it ends up:** `<meta name="description">`, `og:description`, `twitter:description`, BlogPosting JSON-LD `description`.

### `faq-schema-json-2` — single-line FAQPage JSON-LD

**Why this exists as a CMS field:** FAQ rich results are one of the few SERP enhancements Google still rewards generously. Pages with FAQPage schema show expandable Q&A directly in search results — increased CTR, more screen real estate.

**Why per-item and not template:** Different posts have different FAQs. CMS field allows per-post FAQs without copy-paste pollution.

**Anti-pattern:** Pasting `<script type="application/ld+json">` wrapper. The template auto-wraps — including the wrapper produces nested scripts that break parsing.

**How AI engines use it:** FAQ structured data is among the most reliably parsed schema types. AI engines extract Q&A pairs directly from this JSON.

### `main-image` and `thumbnail-image`

**Why dimensions matter:** Open Graph spec requires 1200×630 for full landscape cards. Below that, FB/LinkedIn render a small square thumbnail instead. Twitter's `summary_large_image` card requires similar. Social shares with proper large-format cards get 2-3x more engagement than thumbnail-only.

**Why slug-prefixed filenames:** Webflow Assets is flat across the site. Generic `featured.png` becomes one of 50 such files within months. `{slug}-featured.png` stays uniquely findable.

**Why alt text:** Accessibility + schema.org `image.alt`. AI engines that parse `<img alt="...">` for image-to-text translation prefer descriptive alt over keyword-stuffed or empty alt.

### `author` (reference to Authors collection)

**Why this matters for AEO:** E-E-A-T — Experience, Expertise, Authoritativeness, Trustworthiness. Google's quality rater guidelines explicitly weigh author identity. AI engines increasingly cite "according to {Author Name}" rather than just "according to {Site Name}" — having a real, identifiable, credentialed author makes the citation richer.

**Why a reference field not a string:** Person schema needs structured data (jobTitle, sameAs LinkedIn, worksFor Organization, photo). String would lose all that. Reference connects to Authors collection where each author has full Person profile.

**Why all 4 GreenM authors are real people with photos + LinkedIn + bio + credentials:** Each piece of metadata strengthens the Person schema. Anonymous or stub authors hurt E-E-A-T signal.

### `category` (reference to Categories collection)

**AEO use:** Feeds `articleSection` in BlogPosting JSON-LD + provides topical anchor for content cluster building. AI engines use category to understand topical authority — multiple high-quality articles in same category establishes topical expertise.

### `published-date` + `last-updated` (plus their ISO twin fields)

**Why both fields:** `published-date` is when first published. `last-updated` is when content was meaningfully refreshed. AI engines apply *recency bias* — content older than 6-12 months gets deprioritized. Setting `last-updated` after a real content refresh (not typo fixes) signals freshness without lying about original publication.

**Why ISO twin fields:** Webflow auto-emits `lastPublished` (system timestamp) but the canonical schema.org spec requires `datePublished` and `dateModified` as ISO strings. The `*-iso` PlainText fields let you publish a curated date that's distinct from system timestamps — important when post is published 3 days after first being created.

### `read-time`

**Why string not number:** Display convention. Users see "10 min read" — not a number to compute against.

**AEO secondary effect:** Implicit signal about content depth. Reading time correlates with content thoroughness, which AI engines indirectly value (longer high-quality articles get cited more than 200-word answers).

### `featured` (Switch)

**Why optional:** Conditional Visibility on homepage feature module. Editorial promotion mechanism — most posts default `false`, occasional posts get promoted.

**Not AEO-critical directly**, but featured posts get more internal links and traffic, which feeds the broader authority signal.

## Field-to-AEO-mechanism crosswalk

| AEO mechanism | Fields that feed it |
|---|---|
| AI engine direct citation | `tldr`, `key-takeaways`, `faq-schema-json-2`, body (H2 questions) |
| BlogPosting schema | `name`, `seo-title`, `tldr` (→ description), `published-date-iso`, `last-updated-iso`, `main-image`, `author` ref, `category` ref |
| Person/E-E-A-T schema | `author` ref → Authors collection (name, jobTitle, photo, linkedin, credentials) |
| FAQPage rich result | `faq-schema-json-2` |
| BreadcrumbList | URL structure + `name` |
| Social sharing | `seo-title`, `seo-description`, `main-image` (auto-bound via "Same as SEO title tag" + Open Graph image bind) |
| Search engine indexability | robots toggle (per-item) + Global canonical URL (site-wide) |
| Topical authority | `category` + content cluster of related posts |
| Freshness | `last-updated` set when content meaningfully changes |
| Image SERP | `main-image.alt` + filename + 1200×628 dimensions |

## What's NOT in this plugin

Some AEO infrastructure lives outside individual posts:

- **Organization schema** (sameAs, logo, contactPoint, addresses) — site-wide, emitted on every page from homepage Custom Code. See `Schema/_homepage/webflow-snippet.md`.
- **WebSite schema with SearchAction** — same.
- **llms.txt** — Site Settings → SEO → uploaded once.
- **robots.txt + Allow AI Bots** — Site Settings → SEO → toggle.
- **Global canonical URL** — Site Settings → SEO → field.

If you're publishing the first post on a fresh Webflow site, verify these are in place (most are tracked in GRO-342 and related tickets).

## Background reading

For full project context including current state and active workstreams:

- `Lucy - growth/docs/AEO/AEO-STATUS.md` — single source of truth, updated as work progresses
- Linear epic [GRO-342](https://linear.app/greenminc/issue/GRO-342) — children with detailed sub-tickets per AEO category (A through H + ad-hoc)
