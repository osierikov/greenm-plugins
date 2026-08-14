# Verification — how to measure AEO impact

Load this reference at **Stage 8 (Post-publish)** of the workflow, or whenever the user asks "is this content getting cited?" or "how do we know AEO is working?"

## §1 The honest answer

AEO measurement is immature. There is no Google Search Console equivalent for ChatGPT, Perplexity, or Claude. Verification is a mix of:

1. **Direct observation** — manually testing whether content is cited
2. **Leading indicators** — content principles in Parts 2–6 of the GreenM AEO standard being correctly applied (covered by the pre-publish checklist)
3. **Indirect signals** — referrer logs, traffic patterns, mentions

There is no single number that tells you AEO is "working." Track multiple signals and look for movement over weeks-to-months, not days.

## §2 Manual citation testing protocol

For each piece of content, run this loop:

1. At publish time, define **3–5 query baselines** — the exact natural-language questions a buyer might ask that the content is designed to answer. Log them in the Linear sub-issue for the post.
2. **At T+0** (publish day), run each query through each engine. Record result. This is the baseline.
3. **At T+30 days**, re-run. Record. Compare to baseline.
4. **At T+90 days**, re-run. Record. Compare to T+30.
5. After the 90-day mark, switch to quarterly re-tests unless the content cluster is high-priority.

### Engines to test

- **ChatGPT** (with web browsing enabled — currently the default)
- **Perplexity** (use the free tier; Pro adds models but baseline behaviour is similar)
- **Google AI Overviews** — search Google, look for the AI Overview block at the top of results
- **Gemini** (gemini.google.com)
- **Claude** (claude.ai, with web search enabled)

### Sample baseline queries for GreenM

Use these as starting templates; customise per content piece:

- "What is private AI in healthcare?"
- "How do UK private clinics implement HIPAA-compliant AI?"
- "Who builds agentic AI for healthcare in the UK?"
- "What is the difference between private AI and managed LLM APIs?"
- "How do you integrate AI with Epic / TPP SystmOne / EMIS?"
- "What is GreenM?"
- "Healthcare AI companies in the UK"
- "Private AI for NHS-adjacent organisations"
- "How does FHIR R4 work with clinical AI?"
- "What's the difference between HIPAA-aligned and HIPAA-certified?"

When defining baselines for a specific post, write 3–5 queries that the post is genuinely designed to be the best answer for. Don't over-broaden ("What is AI?") — too competitive. Don't over-narrow either ("What does GreenM's CTO think about FHIR R4 Patient resources?") — too low-volume.

### Logging format (one row per query × engine × date)

Create the Linear sub-issue with a markdown table or attach a sheet:

| Query | Engine | Date | Cited? (Y/N) | Cited passage / position | If no — who was cited? |
|---|---|---|---|---|---|
| "What is private AI in healthcare?" | ChatGPT | 2026-05-01 | N | — | NVIDIA blog, IBM Cloud |
| "What is private AI in healthcare?" | Perplexity | 2026-05-01 | Y | "Private AI is on-premise or VPC-deployed..." (sentence 2) | — |
| "What is private AI in healthcare?" | Google AI Overviews | 2026-05-01 | N | — | Wikipedia, IBM |

Track the **citation rate** = (Y count) / (total tests) over time. Movement from 10% → 30% over a quarter is meaningful.

## §3 What "cited" means

Different engines surface citations differently:

| Engine | Citation behaviour |
|---|---|
| ChatGPT | Inline links in responses when web browsing is used; sometimes a "Sources" section |
| Perplexity | Numbered citations after each sentence; full source list at end |
| Google AI Overviews | Cards with publisher icons on the right side of the overview |
| Gemini | "Sources" expandable section at the bottom |
| Claude | Inline `<cite>` references when web search is used |

Count it as a citation if:
- GreenM is listed as a source, OR
- The answer paraphrases content that's verifiably from a GreenM page

Don't count brand mentions without a source link — that's brand awareness, not AEO citation.

## §4 Indirect signals to watch

Even when direct citation tests are noisy, watch for:

### Referrer traffic
Log entries with these referrers indicate users came from an AI engine:
- `chat.openai.com` / `chatgpt.com`
- `perplexity.ai`
- `gemini.google.com`
- `claude.ai`

In Webflow Analytics or Google Analytics 4, filter the Referrer dimension by these domains. Volume is small but growing — track the trend.

### Direct traffic spikes
Unexplained direct-traffic spikes correlated with content publishing often indicate AI-driven word of mouth — a buyer saw the content surfaced in an AI tool, then visited directly.

### Social / forum mentions
AI engines also pull from Reddit, Hacker News, LinkedIn discussions. Search those platforms periodically for GreenM mentions or paraphrases of GreenM content. Tools like Brand24, Mention, or manual searches work.

## §5 Optional tooling

When manual tracking volume becomes painful (typically >30 active pages being tracked):

- **Profound** — LLM brand monitoring, paid
- **Ahrefs Brand Radar** — tracks LLM mentions, paid (included in Ahrefs Advanced)
- **AthenaHQ** — emerging tool specifically for AEO tracking
- **Otterly.ai** — AEO-focused monitoring

A spreadsheet + Linear sub-issues works fine until volume justifies tooling.

## §6 What to do with the data

After each re-test cycle (T+30, T+90, quarterly):

**If citation rate is rising:** the content is working. Note which engines cite most readily; some content profiles favour Perplexity over ChatGPT.

**If citation rate is flat at zero after 90 days:** revise the content. Likely issues:
- Lead-with-answer is too soft — re-check first 40 words of each H2
- H2s aren't questions in the user's words
- Citations are weak (no dated specific source)
- Content cluster topic is over-saturated by larger publishers

**If citation rate is rising on Perplexity but flat on ChatGPT:** acceptable. They have different retrieval strategies. Perplexity is more permissive about citing smaller publishers; ChatGPT favours established sources.

**If competitor content is cited instead:** read what's being cited and look for the specificity advantage they have. Often it's a single specific stat or section that wins the citation slot.

## §7 Reporting cadence

Recommend to user:
- **Per-post baselines** logged at publish time (Stage 8 deliverable)
- **Monthly aggregate review** — citation rate across all tracked posts, posted in a Linear thread
- **Quarterly content review** — which posts are working, which need revision, which to retire
