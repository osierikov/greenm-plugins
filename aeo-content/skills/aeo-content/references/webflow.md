# Webflow implementation

Load this reference at **Stage 7 (Publish)** of the AEO workflow, when handing the draft to the user for Webflow paste. Also load when configuring schema markup, llms.txt, or internal link structure.

## §1 SEO fields (every CMS item)

Every Blog Post requires these Plain Text fields in Webflow:

| Field | Constraint | Notes |
|---|---|---|
| **SEO Title** | 50–60 chars | Must contain the primary query phrase. Standalone field, not extracted from Body. |
| **SEO Description** | 140–160 chars | Must answer the primary query directly. |
| **OG Image** | 1200×630 | Separate from featured image if a different aspect is needed. |
| **Canonical URL** | optional | Set only if the content exists elsewhere; otherwise omit. |
| **Last Reviewed Date** | required | Updated whenever the post is meaningfully edited. Drives `dateModified` in schema. |

**Webflow limitation:** Reference and Multi-Reference fields cannot bind to native SEO settings. Keep SEO fields as Plain Text on the post itself.

## §1A Content summary fields (every blog post)

These three fields are surfaced separately in the CMS and rendered in different places on the page (Excerpt → card / OG description; TLDR → top-of-article summary block; Key Takeaways → highlights bullet list). They are *not* the same as the body's lead paragraph — they are CMS-level summaries the skill must emit explicitly. The skill MUST output all three in `publish-kit.md` for every greenm.io blog post.

Constraints below are taken directly from the Webflow CMS field hints — match them exactly.

| Field | Type | Constraint | What goes in |
|---|---|---|---|
| **Excerpt** | Plain Text | **140–180 chars, 2–3 sentences** | Used in post card and as fallback for meta description. Also feeds `description` in BlogPosting JSON-LD (`{{ post.excerpt }}` binding). |
| **TLDR** | Rich Text | **40–80 words** | Answer to the post's main question. Shown at top of article (callout block) and used by AI engines as the cited snippet. **Write a complete standalone answer, not a teaser** — if someone reads only this block, they have the answer. |
| **Key Takeaways** | Rich Text | **3–5 standalone bullets, one per line** | Each bullet must make sense quoted out of context — AI engines extract individual lines. Insight-oriented, not topic-oriented. |

Writing rules:

- **Excerpt ≠ TLDR ≠ Key Takeaways.** All three must exist and differ. Excerpt is the social/card hook (curiosity + clarity). TLDR is the answer (declarative, complete). Key Takeaways are the insights (plural, parallel, specific).
- **TLDR is not the first paragraph of the body.** It can rephrase or compress the lead, but the publish-kit must contain it as a separate field. Webflow renders it in a styled callout above the article.
- **Key Takeaways are not section headings.** "How to prepare for CQC framework changes" is a section heading. "Map evidence to the five key questions, not to the 34 quality statements" is a takeaway. Each bullet on its own line. Never sub-bullets, never multi-clause sentences chained with semicolons — one self-contained statement per bullet.
- **The "out-of-context" test for Key Takeaways.** Before sign-off, take any one bullet, quote it without the post title or surrounding bullets, and ask: would a reader who has never seen the post understand what this means? If no, rewrite that bullet. This is the same test AI engines apply when extracting list items for citation.
- **Voice rules still apply.** Run the active voice file's forbidden-words check across all three fields. No hype ("revolutionary," "leverage"), no em dashes, no "Book a demo" CTAs in any of them.
- **AI-citation surface.** Excerpt feeds OG and BlogPosting JSON-LD description. TLDR is the cited snippet for AI Overviews / Perplexity. Key Takeaways are extracted per-line as quotable statements. All three need to be tight, declarative, and free-standing.

## §2 Schema markup templates (JSON-LD)

Webflow has a native **Schema Markup** field in Page Settings (per page) or Site Settings (global). Paste JSON-LD there, or use Custom Code → Inside `<head>`.

### BlogPosting (every blog post)

```json
{
  "@context": "https://schema.org",
  "@type": "BlogPosting",
  "headline": "{{ post.title }}",
  "description": "{{ post.excerpt }}",
  "image": "{{ post.featured-image.url }}",
  "datePublished": "{{ post.published-date }}",
  "dateModified": "{{ post.last-reviewed-date }}",
  "author": {
    "@type": "Person",
    "name": "{{ post.author.name }}",
    "url": "https://greenm.io/authors/{{ post.author.slug }}",
    "sameAs": "{{ post.author.linkedin }}"
  },
  "publisher": {
    "@type": "Organization",
    "name": "GreenM",
    "logo": {
      "@type": "ImageObject",
      "url": "https://greenm.io/logo.png"
    }
  },
  "mainEntityOfPage": "https://greenm.io/blog/{{ post.slug }}"
}
```

### FAQPage (any page with FAQ section)

```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "Does private AI work with our existing EHR?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Yes. GreenM integrates private AI with all major EHRs that expose HL7 v2, FHIR R4, or modern REST APIs..."
      }
    }
  ]
}
```

Add one `Question` block per FAQ pair. Keep the answer text matching what's rendered visibly on the page — do not include facts in schema that aren't in the visible content.

### Service (every service page)

```json
{
  "@context": "https://schema.org",
  "@type": "Service",
  "serviceType": "Private AI Foundation",
  "provider": {
    "@type": "Organization",
    "name": "GreenM",
    "url": "https://greenm.io"
  },
  "areaServed": ["United Kingdom", "United States", "European Union"],
  "audience": {
    "@type": "BusinessAudience",
    "audienceType": "Healthcare organisations, private clinics, NHS-adjacent organisations"
  },
  "description": "Private, HIPAA- and GDPR-compliant AI infrastructure deployed in your VPC or on-premises..."
}
```

### Organization (About page / site-wide)

```json
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "GreenM",
  "url": "https://greenm.io",
  "logo": "https://greenm.io/logo.png",
  "foundingDate": "2014",
  "description": "Healthcare AI infrastructure for UK private clinics and NHS-adjacent organisations.",
  "sameAs": [
    "https://www.linkedin.com/company/greenm/",
    "https://github.com/greenm"
  ],
  "knowsAbout": [
    "Private AI",
    "Agentic AI",
    "FHIR R4",
    "HIPAA compliance",
    "NHS DSPT",
    "Clinical workflow automation"
  ]
}
```

### Article + Quotation (case studies)

Use `Article` for the case-study page itself. For the named client quote, embed `Quotation` schema:

```json
{
  "@context": "https://schema.org",
  "@type": "Quotation",
  "text": "GreenM reduced our survey-to-insight cycle from 14 days to real-time.",
  "spokenByCharacter": {
    "@type": "Person",
    "name": "Jane Doe",
    "jobTitle": "VP Analytics",
    "worksFor": {
      "@type": "Organization",
      "name": "NRC Health"
    }
  }
}
```

## §3 Schema binding limitations — workarounds

**Problem:** Reference fields (like Author → Authors collection) **cannot** be bound inside Webflow's native Schema Markup field.

**Workarounds, in order of preference:**

1. **Render the schema as static HTML** in a Rich Text block at the bottom of the template, wrapped in `<script type="application/ld+json">`. Bind reference fields normally inside the Rich Text. *Trade-off:* schema appears in `<body>` not `<head>` — still valid for Google and AI engines per current docs.
2. **Use Custom Code that reads data-attributes** from the rendered DOM and constructs JSON-LD client-side. *Trade-off:* JS-dependent, some crawlers may not execute.
3. **Keep schema partially static**, binding only Plain Text fields. Hand-fill author and publisher names if they don't change often. *Trade-off:* drift risk.

For new templates, prefer option 1.

## §4 Internal linking discipline

Every blog post must include:
- **2–3 links** to other blog posts in the same content cluster
- **1 link** to a relevant service page
- **1 link** to a case study (if a relevant one exists)

**Anchor text rules:**
- Descriptive and specific
- Reads naturally inside the sentence
- Never "click here," "read more," "this article"

Good examples:
- "Our case study with ROC Clinics on Semble integration"
- "How private AI deployment compares to public LLM APIs for PHI workflows"
- "GreenM's approach to FHIR R4 mapping in clinical workflow automation"

Bad examples:
- "Click here to read more"
- "Our case study" *(too generic)*
- "More info" *(no semantic value)*

## §5 llms.txt (site-level)

Located in **Site Settings → SEO → llms.txt** (Webflow native field).

`llms.txt` is the emerging standard for telling AI engines what a site contains and how it's structured. Keep it lean — a sitemap-like document for AI crawlers.

### Template

```
# GreenM
> Healthcare AI infrastructure for private clinics and NHS-adjacent organisations.
> Private AI Foundation, Agentic AI Systems, Unified Health Data, Clinical
> Workflow Integration, AI Launchpad. Compliance: HIPAA, GDPR, NHS DSPT,
> ISO 27001, FHIR R4.

## Services
- [Private AI Foundation](https://greenm.io/services/private-ai-foundation): HIPAA- and GDPR-compliant private AI deployed in your VPC or on-prem.
- [Agentic AI Systems](https://greenm.io/services/agentic-ai-systems): Multi-step AI agents for clinical workflows.
- [Unified Health Data](https://greenm.io/services/unified-health-data): FHIR R4 ingest, normalisation, and querying across EHR sources.
- [Clinical Workflow Integration](https://greenm.io/services/clinical-workflow-integration): AI-augmented clinical workflows with audit trails meeting DCB0160.
- [AI Launchpad](https://greenm.io/services/ai-launchpad): 8-week structured pilot to production track.

## Case studies
- [NRC Health real-time analytics](https://greenm.io/case-studies/nrc-health): Survey-to-insight time reduced from 14 days to real-time.
- [ROC Clinic](https://greenm.io/case-studies/roc-clinic): Private AI on Semble for UK private clinics.

## Blog
- [Cluster: Private AI in Healthcare](https://greenm.io/blog/category/private-ai-and-compliance)
- [Cluster: Agentic AI in Practice](https://greenm.io/blog/category/agentic-ai-in-practice)
- [Cluster: Healthcare Intelligence](https://greenm.io/blog/category/healthcare-intelligence)

## About
- [About GreenM](https://greenm.io/about-us): Founded 2014, healthcare AI for UK and US.
```

Update llms.txt whenever a new service launches, a case study is published, or content cluster taxonomy changes.

## §6 robots.txt and AI bots

Webflow has a native **AI Bots toggle** in Site Settings → SEO.

**Recommended setting for GreenM: Allow AI bots.** Healthcare buyers research extensively via ChatGPT and Perplexity. Blocking AI crawlers cuts GreenM out of those answers entirely.

Bots allowed (on by default if AI Bots toggle is on):
- GPTBot (OpenAI training)
- ChatGPT-User (OpenAI live retrieval)
- ClaudeBot (Anthropic)
- PerplexityBot
- Google-Extended (Gemini training)
- Bingbot (Microsoft, including Copilot)

If a specific client engagement requires non-indexing of a specific page, set `noindex` on that page, not a global block.

## §7 Rich Text discipline (CMS Designer)

When writing in the CMS Rich Text editor:

- **H2** for top-level questions, **H3** for sub-sections, **H4** sparingly.
- Use the native **Wrap with Span** in the toolbar for emphasis — don't paste HTML.
- **Never click into individual elements** inside CMS-bound Rich Text in Designer to style them. Style cascades through the wrapper class's nested selectors (`All H2`, `All Paragraphs`, etc.) on the Style panel.
- For new pages: duplicate an existing live page rather than building from scratch. This preserves CMS bindings and class structure.

## §8 Performance targets

AI engines deprioritise slow pages. Targets:

| Metric | Target | Notes |
|---|---|---|
| LCP (Largest Contentful Paint) | < 2.5s | |
| CLS (Cumulative Layout Shift) | < 0.1 | |
| Page weight (first load) | < 1.5 MB | |

Webflow generates static HTML — no SSR concerns. Main risk: oversized images. Use Webflow's responsive image variants and serve WebP where possible.

## §9 Stage-7 deliverable checklist (what to hand the user)

When a draft is publish-ready, deliver:

1. **SEO Title** (≤60 chars, contains primary query)
2. **Meta Description** (≤155 chars, answers primary query directly)
3. **Body** (markdown, with H2/H3 as questions, paragraphs 40–80 words)
4. **Schema JSON-LD** (paste-ready, with workaround applied if Reference fields are involved)
5. **Internal link list** (anchor text + target URL × at least 3 links)
6. **Featured image brief** (per GreenM Visual Content Rules v1.1 in Linear)
7. **Author reference** (which Author CMS entry to bind)
8. **Category reference** (which of the 5 CMS categories)
9. **Content cluster tag** (one of the 4 AEO clusters)
10. **Last Reviewed Date** (today)
