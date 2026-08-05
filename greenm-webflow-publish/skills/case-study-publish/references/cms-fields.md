# Case Studies CMS — field schema (greenm.io)

Collection ID `6a5628d1699b78cee3a4502d`. **Always re-read live slugs with `get_collection_details` before assembling `fieldData`** — slugs are not always the display name, and the collection is at 60/60 fields (full). Snapshot below; API is source of truth.

`fieldData` value shapes: Image = `{"fileId","url","alt"}`; Link = URL string; Option = the option **ID**; MultiReference = array of item IDs; Rich Text = single continuous HTML string (no inter-tag whitespace).

## SEO / service
| Field | Slug | Type | Req | Notes |
|---|---|---|---|---|
| Name | `name` | PlainText | yes | internal CMS list name; not on page |
| Slug | `slug` | PlainText | yes | = old static slug (keep URL) |
| Meta Title | `meta-title` | PlainText | yes | `Topic \| Client Case Study \| GreenM`, ~60 |
| Meta Description | `meta-description` | PlainText | yes | 150–160, include key result |
| OG Image | `og-image` | Image | | 1200×630; falls back to Hero |
| Published Date ISO | `published-date-iso` | PlainText | | `2026-07-10T00:00:00Z` (datePublished) |
| Last Updated ISO | `last-updated-iso` | PlainText | | short ISO (dateModified) |

## Listing card
| Field | Slug | Type | Req | Notes |
|---|---|---|---|---|
| Card Summary | `card-summary` | PlainText | yes | 1–2 sentences (= publish-kit Excerpt) |
| Solution Category | `solution-category` | Option | yes | pass option ID (below) |
| Impact 1/2/3 | `impact-1`/`impact-2`/`impact-3` | PlainText | | short result, no trailing period |

Solution Category option IDs: AI MVP `f3698b986a5562c21bc92c4fd811da00` · Private AI `52dcb143a058b34b2e72bcf35ce78f3d` · Unified data `81899337b2bf5ece02581d17b3af3a44` · Workflow integration `985781462a5d097df3c7f42889159db5` · Agentic AI `e284c0b63d55f545c4eb8113047c0585` · Scaling & optimization `d3a1e853a14e46350aa559a185d4332c`.

## Hero
| Field | Slug | Type | Req |
|---|---|---|---|
| Hero Title | `hero-title` | PlainText | yes |
| Hero Subtitle | `hero-subtitle` | PlainText | yes |
| Hero Image | `hero-image` | Image | yes |

## Client
| Field | Slug | Type | Req |
|---|---|---|---|
| Client Name | `client-name` | PlainText | yes |
| Client Logo | `client-logo` | Image | yes (required) |
| Client Description | `client-description` | PlainText | yes |
| Client Link URL | `client-link-url` | Link | yes (required) |
| Client Link Label | `client-link-label` | PlainText | | e.g. "Visit website" |

## Stats (4 slots — value empty = text badge)
| Slot | Value slug | Label slug |
|---|---|---|
| Stat 1 | `stat-1-4-value` | `stat-1-4-label` |
| Stat 2 | `stat-2-value` | `stat-2-label-5` |
| Stat 3 | `stat-3-value` | `stat-3-label-5` |
| Stat 4 | `stat-4-value` | `stat-4-label-5` |

Fully-empty stat cards auto-hide (template-gotchas). A label with no value is a valid badge.

## Quote (client)
| Field | Slug | Type |
|---|---|---|
| Quote Text | `quote-text` | PlainText (no quote marks — design adds them) |
| Quote Author | `quote-author` | PlainText |
| Quote Role | `quote-role` | PlainText |

## What we did
| Field | Slug | Type | Req |
|---|---|---|---|
| What We Did Intro | `what-we-did-intro` | PlainText | yes |
| What We Did Tagline | `what-we-did-tagline` | PlainText | yes |

## Body — Rich Text (one continuous HTML string each)
| Field | Slug | Req | HTML shape |
|---|---|---|---|
| Challenge | `challenge` | yes | `<p>` + optional `<ul>` |
| Challenge Image | `challenge-image` | | Image (optional) |
| How It Was | `how-it-was` | yes | `<ul>` short items |
| Solution | `solution` | yes | `<p>` + `<ul>` |
| Solution Image | `solution-image` | | Image (optional) |
| Capabilities Heading | `capabilities-heading` | yes | default "Key Capabilities" (or "How It Is") |
| Capabilities | `capabilities` | yes | `<ul><li>…</li></ul>` (pills) |
| Outcomes | `outcomes` | yes | intro `<p>` + `<h3>`+`<p>` pairs + optional trailing `<blockquote>` |
| Delivery Intro | `delivery-intro-2` | | optional `<h3>` + `<p>` |
| Delivery Phase 1 | `delivery` | yes | `<h3>` name + `<ul>` |
| Delivery Phase 2 | `delivery-phase-2-2` | yes | one phase |
| Delivery Phase 3 | `delivery-phase-3-2` | yes | one phase |
| Delivery Phase 4 | `delivery-phase-4` | | optional; empty for 3-phase cases |
| Tech Stack | `tech-stack` | | flat `<ul><li>name</li></ul>`; core tech only, no SaaS (e.g. no Front) |
| Security | `security` | yes | intro `<p>` + `<h3>`+`<p>` pairs |
| Security Logos | `security-logos-2` | | MultiReference → Security Logos `6a59f8908cbfa8b087d3baf3` (non-draft) |

Legacy spare slot: `outcomes-quote-text-author-role` (PlainText) — outcomes quote now lives inside `outcomes`.

## Custom sections (optional — hidden if heading empty)
`custom-section-1-heading` / `custom-section-1-body` (RichText) / `custom-section-1-image` (Image); same for `-2-`.

## CTA
| Field | Slug | Type | Req |
|---|---|---|---|
| CTA Heading | `cta-heading` | PlainText | yes |
| CTA Button Label | `cta-button-label` | PlainText | yes (default "Book assessment") |
| CTA Button Link | `cta-button-link` | Link | yes (default Calendly link) |

## Services
| Field | Slug | Type |
|---|---|---|
| Services Provided | `services-provided` | MultiReference → List of Services `6a56326e0d6a2cd9f5f6c910` |
