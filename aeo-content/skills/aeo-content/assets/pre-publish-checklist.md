# Pre-publish checklist

Load this asset at **Stage 6 (Self-review)** of the AEO workflow. Run only the channel-relevant items. Report each item's status to the user as: ✅ passed / ❌ failed / ⚠️ needs user input.

Copy the relevant sections into the Linear sub-issue when Stage 8 starts.

The structure: **Always** → **Web only** → **LinkedIn only** → **Healthcare** (every channel) → **Stage 8** (every channel).

---

## Workspace — every piece, every channel

Provisioned at Stage 2.5. Verify the pair still exists and is intact before publishing.

- [ ] **Drafts folder** exists at the correct channel path (see `references/workspace-provisioning.md` §1)
- [ ] **`brief.md`** inside the folder has `linear_ticket` frontmatter populated
- [ ] **Linear ticket** exists, is in the correct project (Content & Testimonials for web/external/Substack/Other; Social Selling & LinkedIn Content 2025-2026 for LinkedIn), and is assigned to a real person (not empty)
- [ ] **Ticket description** links back to the drafts folder path
- [ ] **Workflow checklist** in the ticket is up to date — every completed stage is ticked

## Always — run for every piece, every channel

These are the AEO floor. Skip none of them, even for a quick LinkedIn post.

- [ ] First lines answer the reader's main question directly (first 40 words for web, first 2 lines for LinkedIn). No preamble.
- [ ] Every paragraph stands on its own (40–80 words for web, 1–3 sentences for LinkedIn). No "as mentioned above" or "see below."
- [ ] Every factual claim has a named, dated source with a specific document/section. No "studies show" or "per NHS guidelines."
- [ ] Real author assigned — name, role, credentials. LinkedIn link in author block (for web) or author profile (for LinkedIn). No "GreenM Team."
- [ ] All acronyms spelled out on first use (HIPAA, GDPR, NHS DSPT, FHIR, PHI, DCB0129, etc.)
- [ ] Voice file from `references/voice/` loaded for the chosen voice — and its rules applied (no hype words from its Tier 1 forbidden list, register matches)
- [ ] Numbers are specific (no "many," "most," "significantly," "vast majority" — use observed counts or labelled estimates)

---

## Web only — blog post, service page, case study, Substack issue, external long-form

Run when the channel produces a long-form piece with headings.

### Content

- [ ] All main headings (H2/H3) are written as questions
- [ ] At least one comparison table or parallel list (compliance frameworks, steps, options)
- [ ] FAQ section with 3–6 question/answer pairs sits before the closing CTA
- [ ] No marketing fluff: no "leverage," "robust" (without grounding), "seamless," "best-in-class," "revolutionary," "cutting-edge"

### Content summary fields (greenm.io CMS)

- [ ] **Excerpt** present, 140–180 chars, 2–3 sentences — reads as social/card hook (curiosity + clarity)
- [ ] **TLDR** present, 40–80 words — complete standalone answer to the post's main question (not a teaser)
- [ ] **Key Takeaways** present, 3–5 standalone bullets, one per line
- [ ] **Out-of-context test:** every Key Takeaway bullet still makes sense when quoted alone, without the post title or surrounding bullets
- [ ] Excerpt, TLDR, and Key Takeaways are three distinct items — none is a copy or near-copy of another
- [ ] All three pass the active voice file's forbidden-words check (no hype, no em dashes)

### Authority (E-E-A-T)

- [ ] Real author photo present
- [ ] Published date set
- [ ] Last reviewed date set (today's date if just edited)
- [ ] Author bio block rendered at bottom of post

### Internal linking (greenm.io only — skip for Substack / external)

- [ ] 2–3 links to other blog posts in the same content theme
- [ ] 1 link to a relevant service page
- [ ] 1 link to a case study (if applicable)
- [ ] All anchor text is descriptive (no "click here," "read more," generic "our case study")

### Technical: SEO + Schema (greenm.io only)

- [ ] SEO Title: 50–60 chars, contains primary query phrase
- [ ] SEO Description: 140–160 chars, answers the primary query directly
- [ ] OG Image: `{slug}-cover.png`, 1200×630 (one file only — `webflow-publish` auto-derives the thumbnail)
- [ ] Canonical URL set only if content exists elsewhere
- [ ] Schema: `BlogPosting` (or `Article` for case studies; `Service` for service pages) applied
- [ ] Schema: `FAQPage` applied if FAQ section exists
- [ ] Schema: Reference-field workaround applied if needed (see `references/webflow.md` §3)
- [ ] Cover image alt text is descriptive (not "image" or filename)

### Image files in `img/` (greenm.io only)

- [ ] **Cover:** exactly one file named `{slug}-cover.png` at 1200×630 (no separate `{slug}-cover-thumbnail.png` — `webflow-publish` derives thumbnail on upload)
- [ ] **In-post schemas** (if any): `{slug}-01.png`, `{slug}-02.png`, … numbered by order of appearance in the body (top-to-bottom)
- [ ] Each schema PNG has a matching `{slug}-NN-embed.html` sidecar if it's meant to render as a Rich Text embed
- [ ] No ad-hoc names (`schema-a.png`, `decision-tree.png`, `figure1.png`) — every file starts with the full slug
- [ ] No legacy `-featured.png` / `-thumbnail.png` files in `img/` — those name conventions are deprecated

### Visual (greenm.io only)

- [ ] Cover image follows GreenM Visual Content Rules v1.1 (Linear)
- [ ] No stock photos
- [ ] No third-party logos in social visuals (rebuild in GreenM style if needed)
- [ ] Teal `#1FD3AF` used only for UI elements, not in blob illustrations

### CTAs (greenm.io blog posts)

- [ ] "Book a Call" CTA present at end of post
- [ ] "Subscribe to GreenM Brief" CTA present at end of post

### Theme discipline (greenm.io)

- [ ] Post is tagged to one of the 4 AEO content themes (Private AI in Healthcare / HIPAA-Compliant AI / EHR + AI Integration / Healthcare AI Without Vendor Lock-In)
- [ ] Post is tagged to one of the 5 CMS categories (Healthcare Intelligence / Agentic AI in Practice / Private AI & Compliance / Inside GreenM / Industry Digest)
- [ ] Theme ≠ category — both assigned

### Substack / external long-form

- [ ] Subject line ≤55 chars (Substack)
- [ ] Preview text ≤90 chars (Substack)
- [ ] One outbound link to a relevant greenm.io page (Substack / external)
- [ ] Anchor text negotiated with host publication (external articles)
- [ ] House-style English variant matched to host (British vs International — external articles)

---

## LinkedIn only

Run when the channel is a LinkedIn post.

- [ ] First 2 lines (visible before "See more") either state the answer directly or pose the question sharply enough to earn the click
- [ ] No question-form headings — LinkedIn doesn't have headings; the rule doesn't apply
- [ ] Paragraphs are 1–3 sentences with blank lines between blocks (mobile readability)
- [ ] Post closes with a question that invites the reader's experience — not "Book a demo" / "Ready to learn more"
- [ ] Outbound link is in the first comment, NOT in the post body (LinkedIn deranks posts with external links in body)
- [ ] Tier 1 source cited in body by name + date (URL goes in first comment)
- [ ] 3–5 hashtags suggested (mix of broad + niche)
- [ ] Image (if any) is 1200×627 LinkedIn-native ratio
- [ ] Posting time suggested (default: Tuesday–Thursday, 9–11am UK time)

---

## Healthcare — every channel

Healthcare guardrails apply regardless of where the content lives.

- [ ] Compliance frameworks named precisely — no "HIPAA" mentioned for UK-only content, no "GDPR" mentioned for US-only deployments
- [ ] No medical advice or clinical recommendations made by GreenM (AI positioned as decision support, not decision maker)
- [ ] Sources cited are Tier 1 (regulators, peer-reviewed journals, standards bodies) or labelled Tier 2 (with methodology noted)
- [ ] No LinkedIn posts (from others), vendor blog posts, or undated stats used as primary sources
- [ ] Data residency claims are specific (region, encryption posture, egress rules) — not just "the cloud" / "private deployment"
- [ ] FHIR claims specify version (R4 / R5) and resource types
- [ ] EHR integration claims name the EHR specifically (Epic via App Orchard, etc.) — no generic "EHR-integrated"
- [ ] Compliance posture uses "aligned with" vs "certified" accurately
- [ ] Client names (NRC Health, NPH Group, Medefer, ROC Clinics, Empower Work, GHX, Quantive Radianse, Human API, Starschema) have explicit publication permission for this piece — confirm with user before naming

---

## Stage 8 readiness — every channel

- [ ] 3–5 query baselines defined for citation testing (drafted in Linear sub-issue, ready to test at T+30 days)
- [ ] Author + last reviewed date + channel confirmed before publish
- [ ] Published URL captured for Stage 8 verification (post URL for LinkedIn / Substack)
