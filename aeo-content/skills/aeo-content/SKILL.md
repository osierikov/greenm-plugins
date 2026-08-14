---
name: aeo-content
description: Create, edit, review, or outline AEO-optimized content for GreenM — a healthcare AI company whose content needs to be cited by ChatGPT, Perplexity, Gemini, Google AI Overviews, and Claude. Use whenever the user wants to write, draft, edit, or plan GreenM content across any channel — blog posts, service pages, case studies, About sections, landing pages, LinkedIn posts, Substack issues, external articles, web copy — even if AEO or SEO is not mentioned. Picks the right voice (Alexey Litvin, GreenM brand, Universal, or Custom) and channel-specific rules.
---

# AEO Content Creation for GreenM

GreenM is a healthcare AI company (greenm.io) targeting UK private clinics and NHS-adjacent organizations. Content must satisfy two audiences simultaneously: human readers (clinical operations leaders, IT directors, compliance officers) AND AI search engines that decide which sources to cite (ChatGPT, Perplexity, Google AI Overviews, Gemini, Claude).

**Conversation rule:** the user works in Ukrainian; all content deliverables are in English. Translate questions and explanations to Ukrainian when talking to the user. Keep prompts plain and human — avoid jargon like "Primary question," "Cluster," "E-E-A-T credentials" when asking the user. Use the simple phrasings shown in the brief below.

## When this skill applies

Apply this skill whenever you are about to:
- Write a new blog post, service page, case study, About section, or any web copy for greenm.io
- Write a LinkedIn post, Substack/newsletter issue, or guest article for an external publication that supports GreenM's positioning
- Edit or refactor existing GreenM content (any channel)
- Review a draft for publication readiness
- Plan a content brief or outline
- Decide internal link structure for new content on greenm.io
- Choose JSON-LD schema for a page on greenm.io
- Evaluate whether existing content gets cited by AI engines

If the conversation is about GreenM content in any form, this skill applies. Don't wait for the user to mention "AEO" — they may just say "write a blog post" or "draft a LinkedIn post."

## The 8-stage content creation workflow

Every piece of content goes through these stages. State which stage you are in when working with the user.

### Stage 1 — Ideation
The user picks a topic. Your job: confirm it maps to one of GreenM's four main themes (load `references/content-types.md` if you need the full descriptions):

- **Private AI in Healthcare** (on-premise, sovereign, no data leaves the clinic)
- **HIPAA-Compliant AI** (US compliance angle)
- **EHR + AI Integration** (FHIR, clinical workflow)
- **Healthcare AI Without Vendor Lock-In** (open standards, portability)

If the topic does not fit any of these themes, surface that explicitly and ask the user whether to (a) reframe the topic, (b) skip the theme mapping for this piece, or (c) add a new theme. Do not silently proceed.

### Stage 2 — Brief
Before drafting, walk the user through these questions, in this order. Use the plain phrasings on the right when speaking to the user; do not paste internal labels.

| Field | What to ask the user (plain language) |
|---|---|
| Reader question | What's the main question the reader has, that this text closes? One sentence. |
| Reader | Who is reading this? One or two phrases (e.g. "clinical director at a 20-clinician UK clinic") |
| Theme | Which of our four themes is this? (offer the list from Stage 1) |
| Channel | Where will this be published? (offer the channel list below) |
| Voice | What style should the text sound like? (offer the four voice options below) |
| Author | Who signs this? Name, role, LinkedIn link |
| Review date | Date of last review (today by default) |

#### Channel options

The channel choice determines which rules apply and how Stage 7 (publish) works.

- **Blog post on greenm.io** — long-form for search and AI citation
- **Service page on greenm.io** — sales-oriented page with concrete capability statements
- **Case study on greenm.io** — client story (named or anonymous)
- **LinkedIn post** — short form, written for humans and indexed by AI engines
- **External article** — guest post / linkbuilding article on a third-party site
- **Substack or email newsletter** — long-form for an existing audience
- **Other** — user describes the channel in their own words; ask one follow-up to lock down format constraints

#### Voice options

The voice choice determines which voice file loads at Stage 5 (draft). Pick exactly one. Never load more than one voice file at the same time — they are mutually exclusive.

- **Alexey Litvin (CEO)** — first-person, diagnostic, pattern observations from delivery experience. Best for thought leadership and CEO-bylined content (LinkedIn posts under Alexey's name, op-eds, Substack issues he writes). Loads `references/voice/alexey-litvin.md`.
- **GreenM as a company** — collective "we," operational grounding, brand voice. Best for blog articles, service pages, case studies, and LinkedIn posts on the company page. Loads `references/voice/greenm.md`.
- **Universal** — neutral, evidence-first, no hype, citation-led baseline. Best for reference content (FAQ pages, glossary, integration docs) and any piece where no specific personal/brand voice is needed. Loads `references/voice/default.md`.
- **Custom** — user provides their own reference (a URL, a pasted text sample, or a description of the desired voice). Loads `references/voice/custom.md` and follows its protocol to extract a voice profile and confirm with the user before drafting.

Do not move to Stage 3 until the seven brief items above are answered.

### Stage 2.5 — Provision workspace

Immediately after Stage 2, before any research or drafting, create a **pair**: a drafts folder on disk and a Linear ticket. Both are always created together and linked to each other. The ticket is assigned to the person who invoked the skill (the initiator).

Load `references/workspace-provisioning.md` for the full protocol — channel → folder mapping, channel → Linear project mapping, initiator resolution (Cowork email → Linear user, with fallback to asking), ticket template with stage checklist, `brief.md` frontmatter, idempotency rules, and failure-mode handling.

Short version: use existing GreenM folders (`content/blog/drafts/{slug}/`, `content/services/drafts/{slug}/`, `content/case studies/drafts/{slug}/`, `content/linkedin/drafts/{YYYY-MM-DD}-{slug}/`, `content/link building/drafts/{host}-{slug}/`, `content/substack/drafts/{slug}/`, `content/other/drafts/{slug}/` — the last two create on first use). Use existing Linear projects: `Content & Testimonials` for every channel except LinkedIn; `Social Selling & LinkedIn Content 2025-2026` for LinkedIn. Do NOT create a new "AEO Content Pipeline" project — that would duplicate existing structure.

Do not move to Stage 3 until both the folder and the Linear ticket exist and are linked (folder has `brief.md` with `linear_ticket` frontmatter field; ticket description contains the folder path). Confirm to the user with a two-line status block before proceeding:

> Workspace provisioned:
> 📁 `content/{channel}/drafts/{slug}/`
> 🎫 `{Linear ticket URL}` — assigned to {initiator name}

### Stage 3 — Research
Use web_search to find Tier 1 sources for citations. Tier 1 = primary regulators and peer-reviewed journals:
- NHS England, NICE, MHRA, ICO (UK)
- HHS, FDA, NIST, AHRQ (US)
- EMA, EDPB (EU)
- HL7, ISO, HITRUST (standards)
- Lancet Digital Health, npj Digital Medicine, JAMIA, BMJ, Nature Medicine, NEJM AI (journals)

Load `references/healthcare.md` for the full source list and citation specificity rules. Every factual claim about regulation, clinical outcomes, or compliance MUST cite a Tier 1 source with date and specific document/section, not just a domain link. This applies to every channel — LinkedIn posts that cite vague sources are just as unciteable as blog posts that do.

### Stage 4 — Outline

For **web channels** (blog post, service page, case study, External article published as a long-form web page, Substack issue): build the outline using question-form headings. Each heading is the literal question a reader (or AI engine) might ask. Examples:

- ❌ "Implementation" → ✅ "How does GreenM deploy Private AI in a clinic?"
- ❌ "Compliance" → ✅ "Which regulations apply when AI processes PHI?"

Sub-questions go in sub-headings. Show the outline to the user before drafting and get explicit approval. For per-format outline shape (where the FAQ block sits, where internal links go), load `references/content-types.md`.

For **LinkedIn posts**: there is no outline in the traditional sense. Instead, draft a 3-line skeleton: hook → main insight → closing question. Show that skeleton to the user before drafting the full post.

### Stage 5 — Draft

Apply the rules in the "AEO rules — what applies where" section below. Load the chosen voice file from `references/voice/` and apply its voice rules on top.

### Stage 6 — Self-review

Before showing the draft to the user, run it through `assets/pre-publish-checklist.md`. The checklist is grouped by channel — only run the items that apply to the chosen channel. Report which items passed, which failed, and which need user input.

### Stage 7 — Publish

Publish flow depends on the channel.

**Blog post / service page / case study on greenm.io:** The user publishes in Webflow. Provide them with:
- SEO Title (≤60 chars), Meta Description (≤155 chars), OG image notes
- **Excerpt** (140–180 chars, 2–3 sentences) — post card preview, fallback for meta description
- **TLDR** (40–80 words) — complete standalone answer to the post's main question, rendered as a callout above the body; this is the snippet AI engines cite
- **Key Takeaways** (3–5 standalone bullets, one per line) — each bullet must make sense quoted out of context; AI engines extract individual lines
- Schema JSON-LD ready to paste into Webflow page-level schema field (`references/webflow.md`)
- Internal link list (anchor text + target URL)
- Featured image brief with **concept rationale** on top (see below — follows GreenM Visual Content Rules v1.1)
- Last reviewed date (CMS field)

Excerpt, TLDR, and Key Takeaways are three **distinct** CMS fields — never collapse them into one. See `references/webflow.md` §1A for the writing rules and the difference between them. All three go into the publish-kit.md as separate rows in the CMS field mapping table.

Load `references/webflow.md` for exact field mappings and Webflow-specific gotchas.

**Two separate image tracks — do not conflate them.** Every post can carry two kinds of visuals, and they are designed and generated *differently*. Never treat them as one job.

- **Track 1 — Featured / OG image (mandatory, one per post):** the hero + Open Graph preview card. Must be **photo-realistic**, no text, no UI, no readable labels — see the `img-for-blog` skill Mode A / B / C rules. This image lives in `main-image` and `thumbnail-image` CMS fields (1200×630 + 1024×536). Always required. Never use a diagram or schema here — social preview cards render at small size, text and UI elements become illegible, and the image looks like a stock infographic rather than a magazine cover.

- **Track 2 — In-post schemas / diagrams (optional, zero-to-many per post):** UI-style illustrations embedded in the body to explain a decision, a transition, a comparison, or a sequence. These use the GreenM design system (typography, palette, buttons, cards — see `greenm-design` skill). They may carry readable text, arrows, branches, tabs. They live inside the post body, not in the featured slot.

### Stage 7a — Image inventory (mandatory scan before any image work)

Before writing the Featured image brief, scan the finished draft for candidates that would benefit from a Track-2 in-post schema. Look specifically for:

1. **Decision tree / yes-no branching** — the draft asks a question and splits into two paths with different consequences.
2. **Framework transition** — old thing → new thing (e.g. "one SAF becomes four sector frameworks"), before/after states.
3. **Structured comparison** — 3+ named entities with parallel attributes (vendors, deployment types, frameworks).
4. **Sequential stages / process steps** — a numbered flow the reader has to hold in their head.
5. **Layered system architecture** — components stacked or connected (data layer, integration layer, presentation layer).

If any pattern is present, list the candidates to the user as a menu. For each, give a one-sentence rationale (which section of the post, what the schema clarifies, why a diagram beats prose here). Then pause for the user to pick: (a) generate this schema, (b) skip it, (c) modify the concept first.

Do not auto-generate schemas — always confirm each one. In-post schemas are optional; a good post can ship with zero. But if you scanned and found no candidates, say so explicitly ("I scanned the draft — no decision trees, transitions, or comparison structures that would benefit from a diagram; proceeding with Featured image only") so the user knows the check was run.

### Stage 7b — Featured / OG image brief

Independent from any Track-2 work. Featured is always required. Follow the `img-for-blog` skill rules — photo-realistic Mode A (interior scene), Mode B (abstract still-life), or Mode C (hybrid interior + unobtrusive screen with no readable text). Never Mode-D or any diagram/schema variant for this slot.

**Legibility rules — enforce before delivering the brief, not after generation:**

The Featured image renders at 1024×536 as a card in feeds and at 1200×630 as OG in previews — the reader sees it for two seconds while scrolling. The brief must produce something that survives that glance. Check every candidate concept against these four gates before writing the prompt:

1. **One focal point.** There is exactly one object the eye lands on first. If two objects compete for weight (same size, same lighting, both centered), the composition fails. Position: centered or on a rule-of-thirds intersection. Everything else in the frame is supporting cast — smaller, softer focus, or in negative space.

2. **≤ 3 symbolic elements total.** Count the metaphor-bearing objects. Envelope + cube + cube-under-dome + connecting wire = four. Too many. Each extra element halves the reader's chance of decoding the metaphor in two seconds. Fewer, larger, more deliberate.

3. **Decodable without caption.** Would a reader who has never seen this post recognise the metaphor from the image alone? If it needs the article to explain itself, the metaphor is too abstract for OG. State the decode explicitly in the rationale: *"The reader will read this as: X."* If you can't finish that sentence in one clause, rework the concept.

4. **Two-second glance test.** Imagine the image scaled to a 200-pixel-wide LinkedIn feed card. Can you still tell what the central object is and roughly what it means? If detail is lost — remove detail from the concept until the essentials survive at that scale.

**Anti-patterns to reject at brief stage** (these look artistically strong in isolation but consistently fail the four gates):

- **Objects under domes / bell jars / glass covers** — adds a layer of visual noise that reads as "specimen display", weakens the object's primacy, and hurts thumbnail legibility.
- **Wires or threads floating in space between multiple objects** — reads as "network diagram in still-life form"; competes with the focal point, distributes attention across the frame.
- **Scattered composition with one different element** — the "spot the odd one out" pattern (10 identical + 1 different). Requires close inspection, which social preview does not give.
- **Sealed envelopes with wax seals** — over-associated with legal/formal correspondence; rarely maps cleanly to modern healthcare/AI themes.
- **Overlapping paper piles with mixed formats** — high element count, low focal-point clarity.

The rationale required below now includes explicit answers to gates 1–4. If any gate fails, do not deliver the brief — revise the concept first.

### Concept rationale — mandatory before any image is shown or briefed

Applies to **both tracks**. Whenever this skill produces an image brief (Track 1 or Track 2) or previews a generated image, prepend a short rationale paragraph. It must answer four questions:

1. **What is the visual metaphor / schema structure?** (e.g. "scattered paper records converging into one filing rack" for Track 1; "two-branch decision tree with 'data crosses boundary' vs 'data stays in' pathways" for Track 2)
2. **Which track and, for Track 1, which mode (A / B / C)?** — one sentence connecting the format to the post's central idea.
3. **How does the visual map to the post's main insight?** — one sentence tying the picture to the reader question from Stage 2.
4. **What was considered and rejected, and why?** — one sentence naming the runner-up concept and why it was set aside (too literal, would date poorly, wrong track for the slot, etc.).

For **Track 1 (Featured/OG)**, add four more answers — the legibility gates from Stage 7b. These are pass/fail; if any fail, do not deliver the brief:

5. **Focal point:** which single object is the eye supposed to land on? (Name it in one word or short phrase.)
6. **Element count:** list the symbolic elements in the frame. Must be ≤ 3.
7. **Decode-without-caption:** finish this sentence in one clause — *"A reader who has not seen this post will read the image as: ..."* If you cannot finish it cleanly, the concept is too abstract.
8. **Two-second thumbnail test:** imagine the image at 200px wide. What survives? If the focal object or the metaphor is unreadable at that size, revise.

The rationale sits in `publish-kit.md` immediately above each image brief as a section titled **"Concept rationale — Featured"** or **"Concept rationale — In-post schema {N}"**. In the Cowork chat, when Claude previews the image (either the brief before generation, or the finished file after cropping), the rationale is stated out loud first — never show the image or the brief without it. This gives the user a decision point before a generation cycle is spent.

**LinkedIn post:** Provide:
- Post body (formatted for LinkedIn: short paragraphs, line breaks between blocks)
- First comment text containing any links and one Tier 1 citation URL (LinkedIn deranks posts with outbound links in the body — put them in the first comment)
- 3–5 hashtags (mix of broad + niche)
- Suggested time-of-day for posting (default: Tuesday–Thursday, 9–11am UK time)
- Image brief if relevant (1200×627 LinkedIn-native ratio)

**External article (guest post / linkbuilding):** Provide:
- Body in the host publication's house style if known (match British vs International English to the host)
- Anchor text suggestions for the agreed link placement
- Bio line for the author block (uses the Stage 2 author field)

**Substack / newsletter:** Provide:
- Subject line (≤55 chars)
- Preview text (≤90 chars)
- Body in markdown
- One outbound link to the relevant greenm.io page if applicable

### Stage 8 — Post-publish

This skill stops at producing the `publish-kit.md`. The actual Webflow publish + post-publish technical verification (GSC, schema, OG, robots, internal links, Lighthouse, Linear closure, rollout log) live in the separate **`greenm-webflow-publish`** plugin, which is the source of truth for that workflow and has been battle-tested on GRO-370.

Two ordered phases after the publish-kit is delivered:

#### Stage 8a — Webflow publish + technical verification (greenm.io channels only)

Hand off to the `greenm-webflow-publish` plugin. Its 10-step workflow reads the `publish-kit.md` this skill produced, uploads images, converts body markdown to Webflow Rich Text, resolves CMS references, creates the CMS item as a draft, promotes it to live, then runs the full validation suite: Google Rich Results Test, schema.org validator, GSC URL Inspection, FB Sharing Debugger, LinkedIn Post Inspector. It also writes the rollout-log row in `Lucy - growth/docs/AEO/Schema/rollout-log.md` and closes the Linear ticket.

If the user expects this skill to do the Webflow publish — surface the boundary explicitly: «The publish step is handled by the `greenm-webflow-publish` plugin. The publish-kit and assets I've produced are the inputs it expects.»

Skip Stage 8a entirely for LinkedIn / Substack / external publications — those channels don't go through Webflow and have no schema / sitemap / GSC.

#### Stage 8b — AEO citation baseline + ongoing measurement (every channel)

Once the post is live (either via `greenm-webflow-publish` for web channels, or manually for LinkedIn / Substack / external), create a Linear sub-issue under the relevant AEO Readiness ticket (GRO-342 children) containing:
- Published URL (or post URL for LinkedIn / Substack)
- 3–5 query baselines to test in ChatGPT / Perplexity / Gemini / Claude / Google AI Overviews 30 days later
- Author + last reviewed date
- Channel

This is how we measure whether AEO actually worked. Load `references/verification.md` for the testing protocol (T+0 baseline, T+30 / T+90 re-tests, quarterly content review).

## AEO rules — what applies where

Rules are grouped by where they apply. Always start with the "Always" block; add the channel-specific block based on the Stage 2 channel choice.

### Always — every piece of content, every channel

These five rules carry the AEO floor. AI engines extract these signals from any text, regardless of where it lives.

1. **Lead with the answer.** The first 40 words (or first 2 lines for LinkedIn) must answer the reader's question directly. No preamble, no "In this article we will explore…" If the answer is buried, the piece doesn't get cited.
2. **Self-contained paragraphs.** Each paragraph stands on its own — 40–80 words for web, 1–3 sentences for LinkedIn. Avoid "as mentioned above" or "see below" — restate. AI engines extract chunks; chunks must make sense out of context.
3. **Specific sources with dates.** Every factual claim about regulation, clinical outcomes, or compliance cites: source name + publication or update date + specific document/section. Example: "NHS England's 2024 *Clinical risk management standard DCB0129*" — not "per NHS guidelines." Vague citations don't get cited.
4. **Named author with role.** Author shows: name, role, credentials where relevant (e.g. "Chief Medical Officer, MBChB, MRCGP"), LinkedIn link. Generic "GreenM Team" bylines fail this. For LinkedIn the author is the profile owner; the role/credential should be in their headline.
5. **Spell out acronyms on first use.** HIPAA = Health Insurance Portability and Accountability Act (US, 1996). GDPR = General Data Protection Regulation (EU, 2018). NHS DSPT = NHS Data Security and Protection Toolkit (UK). FHIR = Fast Healthcare Interoperability Resources. PHI = Protected Health Information. Etc.

### Web only — blog post, service page, case study, Substack issue, external long-form

Add these on top of the Always block when the channel produces a long-form web page with headings.

6. **Question-form headings.** Each main heading is a question a reader (or AI engine) might ask verbatim. Sub-headings carry sub-questions. This aligns with how retrieval models score relevance.
7. **Lists for parallel content.** Compliance frameworks, deployment steps, comparison points — these belong in bulleted or numbered lists. AI engines preferentially extract list items.
8. **FAQ block before the close.** Every long-form piece ends with a 3–5 question FAQ block. On greenm.io that block is marked up as FAQPage schema (see `references/webflow.md`). On Substack / external long-form there is no schema, but the FAQ structure still helps citation.
9. **Schema markup (greenm.io only).** BlogPosting + FAQPage for blogs; Service + Organization for service pages; Article for case studies. Paste-ready templates in `references/webflow.md`.

### LinkedIn only

Add these on top of the Always block when the channel is LinkedIn.

10. **Hook in the first line.** LinkedIn shows only the first 2–3 lines before "See more." That window has to either state the answer (per rule 1) or pose the question sharply enough to earn the click. Question-form headings (rule 6) don't apply.
11. **Short paragraphs.** 1–3 sentences each, blank line between blocks. Mobile rendering eats long blocks.
12. **Closing question, not CTA.** Close with a question that invites the reader's experience ("What's been your biggest integration headache?") not a transactional CTA ("Book a demo"). Engagement signals raise reach; AI engines also see the comment activity.
13. **Outbound links in the first comment.** LinkedIn deranks posts with external links in the body. Cite Tier 1 sources in the body by name + date, put the actual URL in the first comment.

### Voice (always — every piece, every channel)

The active voice is set by the Stage 2 voice choice. Load the matching file from `references/voice/` at Stage 5 and apply its rules. The five Always rules still hold across every voice — voice files only refine *how* this is expressed, they do not loosen the Always rules.

## Decision tree: which references to load

Load reference files only when needed — keep context lean.

- **Always**: this SKILL.md is enough for Stage 1 and Stage 2.
- **`references/workspace-provisioning.md`**: load at Stage 2.5 (provision workspace) — mandatory for every new piece. Contains the channel → folder mapping, channel → Linear project mapping, initiator resolution protocol (email → Linear user with fallback), ticket template, `brief.md` frontmatter, and idempotency rules for resumed sessions.
- **`references/content-types.md`**: load at Stage 4 (outline) once channel + content type is decided. Contains per-channel outline templates: blog post (3 sub-types), service page, case study, About section.
- **`references/healthcare.md`**: load at Stage 3 (research) for any healthcare-specific compliance, regulation, or clinical claim. Also load when drafting service pages or case studies.
- **`references/voice/<choice>.md`**: load at Stage 5 (draft) once voice is decided. Exactly one of: `alexey-litvin.md`, `greenm.md`, `default.md`, `custom.md`. Never load more than one — they are mutually exclusive.
- **`references/webflow.md`**: load at Stage 7 (publish) **only when the channel is on greenm.io**. Contains Webflow CMS field mappings, JSON-LD schema templates (BlogPosting, FAQPage, Service, Organization), internal linking discipline, llms.txt, AI bots toggle. Skip this load for LinkedIn / Substack / external channels.
- **`references/verification.md`**: load at Stage 8b (AEO citation baseline) or whenever the user asks "is this getting cited?" or "how do we measure AEO?" The Webflow publish + 10-step technical verification is owned by the `greenm-webflow-publish` plugin; do not duplicate its protocol here.
- **`assets/pre-publish-checklist.md`**: load at Stage 6 (self-review) always. Run only the channel-relevant items.

## Healthcare-specific guardrails

GreenM operates in a regulated industry. These are not optional and apply across every channel.

1. **No medical advice.** Never write content that could be construed as patient-facing medical advice. GreenM serves providers and clinics, not patients. If the user asks for patient-facing content, surface this concern and ask for confirmation.

2. **Compliance claims need citations.** Any claim about HIPAA, GDPR, NHS DSPT, DCB0129, DCB0160, DTAC, ISO 27001, SOC 2, or HITRUST compliance must cite the specific framework version and section. "HIPAA-compliant" alone is meaningless without scope (which Safe Harbor controls, which sections of 45 CFR §164, etc.). This applies even on LinkedIn — vague compliance claims in a LinkedIn post are still uncited claims.

3. **Data residency claims are jurisdictional.** UK content must reference UK GDPR + Data Protection Act 2018. EU content references EU GDPR + EDPB guidance. US content references HIPAA + HHS guidance. Don't conflate.

4. **FHIR / HL7 / clinical standards.** When discussing EHR integration, specify the FHIR version (R4 / R5) and resource types. Generic "FHIR-compatible" claims are weak.

5. **Client names in case studies and posts.** All GreenM client names that appear in marketing (NRC Health, NPH Group, Medefer, ROC Clinics, Empower Work, GHX, Quantive Radianse, Human API, Starschema) must have explicit publication permission. When drafting any content that names a client — including LinkedIn posts — confirm with user before naming the client.

## Webflow-specific reminders (web-channel only)

These come up at Stage 7 for greenm.io publications. Skip if the channel is LinkedIn / external / Substack.

- Rich Text styling cascades from the wrapper class. Never click individual elements in CMS-bound rich text.
- Reference fields require two-level binding.
- Featured Switch should drive Conditional Visibility, not duplicate fields.
- Schema markup goes in the native Page Settings → SEO → Schema field (page-level) or via embed for site-wide.
- Static HTML output — no SSR issues to worry about.
- Reference / multi-reference CMS fields are NOT supported in schema binding — render those fields in HTML and reference them indirectly.

For full Webflow implementation, including paste-ready schema JSON-LD per content type, load `references/webflow.md`.

## Pre-publish checklist (load `assets/pre-publish-checklist.md`)

Before signing off a draft, run the channel-relevant items. Report which items passed, which failed, and which need user input. Don't tell the user the draft is ready until every applicable item is green or explicitly waived with rationale.

## Integration with existing GreenM tooling

- **Linear AEO Readiness project (GRO-342)** covers the technical/infrastructure side: schema, llms.txt, sitemap, robots.txt AI Bots toggle, Webflow setup. This skill covers the content-production side across channels. They are complementary — cross-reference both when relevant.
- **Visual Content Rules v1.1 (Linear)** govern featured images. Load when designing visuals at Stage 7.
- **CMS schema** for Blog Posts (14 fields), Authors (5 fields, E-E-A-T compliant), Categories (5 fields) is fixed. Field-to-content mapping is in `references/webflow.md`.
- **Theme mapping** is mandatory at Stage 1. Five categories exist (Healthcare Intelligence, Agentic AI in Practice, Private AI & Compliance, Inside GreenM, Industry Digest) but four content themes drive AEO targeting. Don't conflate categories with themes.

## When the user pushes back

Sometimes the user will want to skip rules ("just write the post fast"). Comply, but surface the trade-off clearly in plain language. Example: "Skipping the FAQ block will save 200 words but loses the FAQ schema citation surface — AI engines often pull from that block. Confirm proceed?" Make the cost visible — don't silently drop rules.

## Output format

Drafts are delivered in markdown unless the user requests otherwise. The exact deliverables depend on the channel — see Stage 7. Always include:
- Final title (or first line / hook for LinkedIn) and meta description (web only) at top
- The body
- Pre-publish checklist results
- For greenm.io: Schema JSON-LD block at the end, internal link list, Webflow CMS field mapping
- For LinkedIn: first-comment text, hashtags, posting time suggestion
- For external publication: anchor text suggestions, author bio line
- For Substack: subject line, preview text
