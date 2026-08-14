# Custom voice — user-provided reference

Use this file's instructions when the user selects "Custom" voice at Stage 2. The user provides their own voice reference in one of three forms; this file tells you how to handle each.

## How the user can supply a custom voice

At Stage 2 (Brief), the user picks "Custom" and then provides ONE of:

1. **A URL** — link to a webpage, LinkedIn profile, blog, Substack, or any public text that demonstrates the voice
2. **A pasted text sample** — one or more paragraphs of prose written in the target voice
3. **A descriptive brief** — a short description of the desired voice ("write like a skeptical academic", "warm and conversational, no jargon", "operations-leader peer-to-peer")

You must surface what was provided and confirm understanding before drafting.

## Step-by-step protocol

### Step 1 — Detect and acknowledge the input form

Tell the user explicitly which of the three forms you received, e.g. "You provided a URL. I will fetch and analyze it before drafting." This prevents silent mismatches.

### Step 2 — Extract voice characteristics

Regardless of input form, distill the voice into these dimensions before drafting. Show this analysis back to the user for approval:

- **Register** — formal / business-casual / casual / academic / technical
- **Stance** — declarative / hedged / diagnostic / advocacy / explanatory
- **Sentence rhythm** — short and clipped / varied / long-and-flowing
- **Pronoun and POV** — first person singular ("I") / first person plural ("we") / observational third person / no POV pronouns
- **Forbidden language** — anything you can infer the voice avoids (e.g., no hype words, no em-dashes, no exclamation marks, no jargon)
- **Preferred patterns** — phrases, openings, closers, transition styles that recur in the sample
- **Authority source** — personal observation / collective experience / cited research / categorical claim

If the user provided a URL, fetch and read the page first. If the URL is paywalled, dynamic-rendered, or otherwise unreadable, tell the user and ask for a pasted text sample instead.

If the user provided only a descriptive brief without examples, ask one clarifying question: "Do you have a 1–2 paragraph example I can calibrate against, or should I infer purely from your description?" Inferring purely from description is allowed but lower-confidence — flag this caveat in the draft hand-off.

### Step 3 — Lock the voice profile

Present the distilled profile to the user as a bulleted summary and ask: "Does this match the voice you have in mind? Any corrections before I draft?" Wait for explicit confirmation. Do not proceed to Stage 5 (Draft) without it.

### Step 4 — Draft against the profile

When drafting, keep the profile visible in your working context. After each section, mentally check: would this paragraph pass as written by the voice in the original sample/URL? If not, rewrite before continuing.

### Step 5 — Hand-off with voice notes

When delivering the draft, include a short "Voice notes" block describing which characteristics you applied and where you made judgment calls. This makes it cheap for the user to spot drift and correct.

## Universal AEO rules still apply

Custom voice does **not** override the universal AEO rules from SKILL.md:

- Lead with the answer (Rule 1)
- Question-form H2s (Rule 2)
- Self-contained 40–80-word paragraphs (Rule 3)
- Citation specificity (Rule 4)
- E-E-A-T author attribution (Rule 5)
- Define acronyms on first use (Rule 6)
- Lists for parallel content (Rule 7)
- FAQ block before CTA (Rule 8)

Custom voice changes Rule 9 (Voice and tone) only. If the user's chosen voice would violate any of Rules 1–8 (e.g., a stream-of-consciousness style that breaks paragraph structure), surface the conflict explicitly and ask which to prioritize. Default is to keep Rules 1–8 and approximate the voice within those constraints.

## When custom voice doesn't work

If the user's reference is too short, too generic, or too inconsistent to extract a usable profile, say so directly: "The reference you provided isn't dense enough for me to lock a voice profile. I can either (a) ask for a longer sample, (b) fall back to Default voice with a few of your stated preferences applied, or (c) use one of the built-in voices (Alexey Litvin / GreenM brand) as a starting point and adjust from there." Let the user pick.
