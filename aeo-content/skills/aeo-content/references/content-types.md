# Content type templates

Load this reference at **Stage 4 (Outline)** of the AEO workflow, once the content type is decided. Use the matching template for the brief.

## Quick selector

| User says... | Template |
|---|---|
| "Write a blog post" | §1 Blog posts (pick sub-type below) |
| "Long-form article" / "deep dive" | §1.1 SEO long-form (1,500–2,500 words) |
| "Opinion piece" / "perspective" | §1.2 Thought leadership (600–900 words) |
| "Weekly digest" / "news roundup" | §1.3 Industry digest (400–700 words) |
| "Service page" / "landing page" | §2 Service pages |
| "Case study" / "client story" | §3 Case studies |
| "About page" | §4 About / company pages |

---

## §1 Blog posts

**Target audience.** Decision-makers and practitioners at UK private clinics, NHS-adjacent organisations, and US healthcare AI buyers.

**Length by sub-type.**
- §1.1 SEO long-form: 1,500–2,500 words
- §1.2 Thought leadership: 600–900 words
- §1.3 Industry digest: 400–700 words

**Mix target (Sierikov's content plan):** 60% SEO long-form, 25% thought leadership, 15% digest.

### §1.1 Structure template (SEO long-form)

```
H1: [Question or specific claim]

[TLDR — 40–80 words, complete standalone answer to the post's main question (this is the snippet AI engines cite)]
[Key Takeaways — 3–5 standalone bullets, one per line, each must make sense quoted out of context]

[Author block: name, role, photo, LinkedIn, date]

H2: [Question — what is X?]
   Self-contained answer, 40–80 words.
   Optional: definition list, numbered facts.

H2: [Question — why does X matter?]
   Specific stats, cited sources.

H2: [Question — how does X work?]
   Numbered steps OR labelled sections.

H2: [Question — what are the trade-offs?]
   Comparison table.

H2: [Question — how does GreenM approach X?]
   Specific methodology, named projects/clients (with consent).

H2: Frequently asked questions
   3–6 H3 question/answer pairs.
   Wrapped in FAQPage schema.

H2: Sources and further reading
   Numbered list of primary sources.

[Author bio block]
[Two CTAs: Book a Call + Subscribe to GreenM Brief]
[Related posts — 3 internal links to the same content cluster]
```

### §1.2 Structure (thought leadership, shorter)

```
H1: [Provocative question or specific claim]

[TLDR — 40–60 words, complete standalone answer]
[Key Takeaways — 3 standalone bullets, one per line, each makes sense out of context]

H2: [The observation or problem]
H2: [The insight or counter-argument]
H2: [Why this matters for healthcare AI buyers]
H2: [What GreenM does differently]

[Author bio block]
[CTA: Book a Call]
[Related posts — 2 internal links]
```

### §1.3 Structure (industry digest)

```
H1: [Time period] in Healthcare AI: [Theme of the week]

[40-word opener — what changed this week]

H2: [Headline development #1]
   2–3 sentence explainer + source link
H2: [Headline development #2]
H2: [Headline development #3]
H2: What this means for clinics

[CTA: Subscribe to GreenM Brief]
```

### Schema for blog posts

`BlogPosting` (preferred over `Article` for blog content), plus `FAQPage` for the FAQ section. Both go in Webflow page-level schema field — see `webflow.md` §schema-templates.

### Required CMS fields (Webflow Blog Posts collection)

Title, Slug, **Excerpt** (140–180 chars, 2–3 sentences — post card + meta description fallback), **TLDR** (40–80 words — complete standalone answer; cited snippet for AI engines), **Key Takeaways** (3–5 standalone bullets, one per line, each must make sense out of context), Body, Cover Image (`{slug}-cover.png`, 1200×630 — `webflow-publish` auto-thumbnails to 1024×536 on upload), Category, Author (Reference), Published Date, Last Reviewed Date, Read Time, SEO Title (≤60 chars), SEO Description (≤155 chars), Canonical URL.

The skill MUST emit Excerpt, TLDR, and Key Takeaways as separate rows in the publish-kit.md CMS field mapping table. They are three distinct fields, not one — see `webflow.md` §1A for writing rules and the Excerpt-vs-TLDR-vs-Takeaways distinction.

### Content cluster discipline (MANDATORY at Stage 1)

Every blog post must belong to one of GreenM's four AEO content clusters:

1. **Private AI in Healthcare** — sovereignty, on-premise, no vendor lock-in
2. **HIPAA-Compliant AI** — US compliance angle
3. **EHR + AI Integration** — FHIR, clinical workflow
4. **Healthcare AI Without Vendor Lock-In** — open standards, portability

**Note:** The cluster is different from the CMS category. Categories (Healthcare Intelligence, Agentic AI in Practice, Private AI & Compliance, Inside GreenM, Industry Digest) are UI-level taxonomy. Clusters are AEO targeting. A post has one category AND one cluster.

**Internal-link rule:** at least 2 other posts in the same cluster + 1 service page.

---

## §2 Service & landing pages

**Goal of the page.** Be cited when someone asks ChatGPT "Who does private AI for UK healthcare?" or "What is agentic AI for clinics?"

### Structure template

```
H1: [Service name in plain English]

[Hero paragraph: what it is, who it's for, what outcome — 40–80 words]

H2: What is [service]?
   Definition + scope, 60–100 words.
   This is the highest-yield AEO paragraph on the page.

H2: Who is [service] for?
   Specific user profiles + organisation types.
   "UK private clinics with 5–50 clinicians processing PHI"
   beats "healthcare organisations."

H2: How does [service] work?
   Numbered phases or labelled steps.
   Concrete deliverables per phase.

H2: What compliance does [service] cover?
   List: HIPAA, GDPR, NHS DSPT, ISO 27001, FHIR, etc.
   One line per framework explaining how the service satisfies it.

H2: What's included vs. not included?
   Two-column list. Buyers (and AI engines) need explicit scope.

H2: How long does it take?
   Realistic timeline. Ranges are fine if defined.

H2: How is [service] different from [common alternative]?
   Comparison: managed cloud LLM vs private AI deployment, etc.

H2: Frequently asked questions
   3–6 buyer questions answered concretely.

[Case study cards — 2–3 client examples]
[Trust signals — compliance badges, client logos with permission]
[CTA — Book a discovery call]
```

### Schema for service pages

`Service` with `provider`, `serviceType`, `areaServed`, `audience`. Plus `Organization` for GreenM. Templates in `webflow.md`.

### GreenM's current service pages

- Private AI Foundation (live)
- Unified Health Data (live)
- Clinical Workflow Integration (live)
- AI Launchpad (live)
- Agentic AI Systems (WIP — content needs replacement)

---

## §3 Case studies

Case studies are the highest-trust content type for B2B healthcare buyers — and a very high-yield AEO target because they contain specific outcomes that AI engines love to cite.

### Structure template

```
H1: [Specific outcome] — [Client name]
    Example: "How NRC Health Reduced Survey-to-Insight Time
    from 14 Days to Real-Time"

[Hero summary: 40–80 words — client, problem, solution, outcome with numbers]

H2: About [client]
   Type of organisation, size, geography, what they do.

H2: The challenge
   Specific problem in concrete terms.
   Quantify the cost of not solving it where possible.

H2: The solution
   What GreenM built, in plain language.
   Tech stack as a list (cite specific tools: AWS HealthLake, Snowflake,
   LangGraph, etc.).

H2: How we built it (phases)
   Numbered phases with duration.
   Specific deliverables per phase.

H2: Outcomes
   Quantified results. Numbers, percentages, time saved.
   At least one direct quote from a named client contact (with role).

H2: Compliance and security
   Frameworks, data residency, audit posture.

H2: What's next
   Roadmap or expansion plans.

[Sidebar / fact box: Industry, Team size, Tech stack, Compliance,
 Duration, Engagement type]

[CTA: Book a discovery call]
```

### Non-negotiables

**Numbers.** A case study without specific metrics ("14% faster," "£280K saved," "from 14 days to real-time") is invisible to AEO. Generic case studies get ignored. If the user doesn't have numbers, push back before writing.

**Quote pattern.** One direct quote from the client, with full attribution: name, role, organisation. Wrap in `<blockquote>` and `Quotation` schema, or include in `Article` schema as `mentions`.

**Client permission.** GreenM's client roster (NRC Health, NPH Group, Medefer, ROC Clinics, Empower Work, GHX, Quantive Radianse, Human API, Starschema) needs explicit publication permission for case studies. **Always confirm with user before naming a client.**

### Schema for case studies

`Article` (treat as in-depth report) + `Organization` for both GreenM and the client (with the client's consent).

### Current live case studies

- NRC Health
- ROC Clinic

---

## §4 About / company pages

Lower volume of AEO queries hit About pages, but they're extracted heavily for queries like "Who is GreenM?" or "Is GreenM trustworthy?"

### Must include

- Founding year (2014), headquarters, where the team works from
- Specific industries served (healthcare AI, UK private clinics, NHS-adjacent)
- Specific compliance frameworks (HIPAA, GDPR, NHS DSPT, FHIR, ISO 27001)
- Named leadership team with roles, photos, LinkedIn
- Specific clients (with permission)
- Specific frameworks/methodologies GreenM uses
- Concrete commitments — not vibes

### What to avoid

- "We're passionate about healthcare" → cut
- "Cutting-edge AI" → cut, replace with specific tech stack
- Stock photos of doctors → cut, use team photos
- "Years of experience" without numbers → cut, use "Operating since 2014, 11 years in healthcare AI"

### Schema for About page

`Organization` with `foundingDate`, `numberOfEmployees`, `address`, `sameAs` (LinkedIn, GitHub, etc.), `knowsAbout`.

### Current status

About Us has an HTML prototype only — not in production yet.
