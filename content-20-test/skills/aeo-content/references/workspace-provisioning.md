# Workspace provisioning — folder + Linear ticket pair

Load this reference at **Stage 2.5 (Provision workspace)** — runs once, immediately after the Stage 2 brief is complete and before Stage 3 (research). Its job: create a **pair** — a drafts folder on disk plus a Linear ticket — so every piece of content has a home and an owner from the very start.

Both items are always created together. If either fails, do not proceed to Stage 3 until the failure is surfaced and the user has decided how to handle it. Never leave one side of the pair created without the other.

## §1 Channel → folder path mapping

Folders follow the existing GreenM `Lucy - growth/docs/content/` structure. Use existing folder names verbatim — including spaces where they exist — do not create parallel folders with kebab-case variants.

| Channel (Stage 2 label) | Drafts folder path |
|---|---|
| Blog post on greenm.io | `content/blog/drafts/{slug}/` |
| Service page on greenm.io | `content/services/drafts/{slug}/` |
| Case study on greenm.io | `content/case studies/drafts/{slug}/` |
| LinkedIn post | `content/linkedin/drafts/{YYYY-MM-DD}-{slug}/` |
| External article | `content/link building/drafts/{host}-{slug}/` |
| Substack or email newsletter | `content/substack/drafts/{slug}/` |
| Other | `content/other/drafts/{slug}/` |

Rules:

- **Slug** is kebab-case, lowercase, derived from the post title. Strip stop-words (`the`, `a`, `for`, `to`, `of`) if the raw title is long. Max 60 chars.
- **LinkedIn date prefix** — `YYYY-MM-DD` of the intended publish date (from Stage 2 brief). LinkedIn posts on the same theme recur; the date guarantees uniqueness and preserves chronology.
- **External host prefix** — the domain of the host publication (e.g. `techcrunch-`, `hitconsultant-`). Same external post may be pitched to multiple sites; the prefix keeps them straight.
- **Substack and Other folders don't exist yet** — the skill creates them (with `drafts/` subfolder) on first use. Do not create them proactively.
- **`content/linkedin/briefs/` is not owned by this skill** — it belongs to a separate briefing workflow. Do not write there.
- **The corresponding `final/` folder is not created by this skill** — `greenm-webflow-publish` (or the person publishing) copies the draft into `{channel}/final/{slug}/` at publish time. This skill only touches `drafts/`.

## §2 Channel → Linear project mapping

Use existing Growth-team projects. Do not create new projects.

| Channel | Linear project | Project ID |
|---|---|---|
| Blog post on greenm.io | Content & Testimonials | `54095f04-b1dd-471b-ae17-4d597872220b` |
| Service page on greenm.io | Content & Testimonials | `54095f04-b1dd-471b-ae17-4d597872220b` |
| Case study on greenm.io | Content & Testimonials | `54095f04-b1dd-471b-ae17-4d597872220b` |
| External article | Content & Testimonials | `54095f04-b1dd-471b-ae17-4d597872220b` |
| Substack or email newsletter | Content & Testimonials | `54095f04-b1dd-471b-ae17-4d597872220b` |
| Other | Content & Testimonials | `54095f04-b1dd-471b-ae17-4d597872220b` |
| LinkedIn post | Social Selling & LinkedIn Content 2025-2026 | `b82f2c88-46d7-4bef-84fb-692cc6704e0c` |

Team is always **Growth** (`7c951f16-b671-48aa-9dc8-685a7262fbe8`). Verify project IDs are still current with `list_projects team="Growth"` if you suspect they've moved.

## §3 Initiator (assignee) resolution — three-step fallback

The Linear ticket must be assigned to the person who invoked this skill. Resolve in this order — first success wins.

### Step 1 — Cowork user email

Read the initiator's email from the Cowork session context (usually surfaced in the system prompt as `Email address: xxx@greenm.io`). This is the person actually running the skill in their Cowork.

### Step 2 — Map email to Linear user

Call `list_users query="{email}"` in the Linear MCP. If exactly one match is returned, use its `id` as the assignee.

Known GreenM email → Linear user mapping (cache; verify if user set changes):

| Email | Linear name |
|---|---|
| `alexey@greenm.io` | Alexey Litvin |
| `osierikov@greenm.io` | Sierikov |
| *(others resolved on demand)* | *(query at runtime)* |

If the query returns zero users or multiple, do not silently pick — proceed to Step 3.

### Step 3 — Fallback: ask the user

If steps 1 or 2 fail (no email in context, user not found in Linear, ambiguous match) — ask in plain language: *"Who should own this ticket in Linear? (name or Linear username)"* Then run `list_users query="{answer}"` and use the result. Do not assume Alexey by default — that would silently misroute tickets when someone else is running the skill.

## §4 Ticket structure

### Title

Use the working post title from Stage 2 brief. Prefix with channel if it helps at-a-glance sorting; skip if the title already implies it.

- Blog: `{Post title}`
- Service page: `Service page — {Service name}`
- Case study: `Case study — {Client name or anonymized descriptor}`
- LinkedIn: `LinkedIn post — {Hook or theme}`
- External: `External article — {Host} — {Working title}`
- Substack: `Substack — {Issue title}`
- Other: `{Descriptor} — {Working title}`

### Description (markdown)

Template — populate every section from Stage 2 brief. Use literal newlines, not escape sequences, when saving via `save_issue`.

```markdown
## Brief

- **Reader question:** {main question this content closes}
- **Reader:** {audience — one or two phrases}
- **Theme:** {one of four AEO themes}
- **Channel:** {channel label from Stage 2}
- **Voice:** {Alexey Litvin / GreenM as a company / Universal / Custom}
- **Author byline:** {name, role, LinkedIn}
- **Review date:** {YYYY-MM-DD, today by default}

## Working folder

`Lucy - growth/docs/{drafts folder path from §1}`

## Workflow status

- [x] Stage 1 — Ideation
- [x] Stage 2 — Brief
- [x] Stage 2.5 — Provision workspace (this ticket + folder created)
- [ ] Stage 3 — Research (Tier-1 sources)
- [ ] Stage 4 — Outline
- [ ] Stage 5 — Draft
- [ ] Stage 6 — Self-review
- [ ] Stage 7 — Publish-kit ready
- [ ] Stage 8a — Publish (handoff to `greenm-webflow-publish` plugin for web channels)
- [ ] Stage 8b — AEO citation baseline logged

## Notes

_Skill will append notes on each stage as it progresses._
```

The checklist mirrors the `aeo-content` 8-stage workflow. Each stage should tick its box (via `save_issue patch` or `save_comment`) as the skill progresses. This gives the initiator a passive audit trail — they see progress in Linear without asking.

### Labels

Add a channel label so filtering works. Existing labels — reuse where they exist; do not create new labels proactively.

- Blog / Service / Case study / External / Substack / Other → look for existing `content-web` or similar; if none, leave empty and flag
- LinkedIn → `linkedin` if that label exists in the project

### Priority

Default to **Medium** (3) unless the Stage 2 brief indicates otherwise (e.g. "for the Q4 launch" → High; "evergreen backfill" → Low).

## §5 Folder scaffold

At Stage 2.5, after creating the folder, drop a single `brief.md` inside it. This is the persistent record of the brief that any future session (this one or another) can reload.

```markdown
---
title: {Post title}
slug: {slug}
channel: {channel label}
theme: {theme}
voice: {voice option}
author: {byline}
review_date: {YYYY-MM-DD}
linear_ticket: {Linear URL from §4}
initiator: {email or Linear user}
created_at: {YYYY-MM-DDTHH:MM:SSZ}
---

# Brief — {Post title}

## Reader question

{main reader question — one sentence}

## Reader

{audience — one or two phrases}

## Voice reference

Loaded at Stage 5: `references/voice/{voice-slug}.md`

## Sources on hand

{any pre-known Tier-1 sources the user mentioned at brief time}
```

Later stages write additional files into the same folder:

- `outline.md` (Stage 4)
- `blog_{slug}_{YYYY-MM-DD}_v1.md` (Stage 5 — draft, versioned)
- `publish-kit.md` (Stage 7)
- `{slug}-featured.png`, `{slug}-thumbnail.png` (external — from `img-for-blog` skill)

## §6 Ordering guarantees

Provision the pair in this exact order:

1. **Compute** the folder path from channel + slug (§1).
2. **Compute** the Linear project ID from channel (§2).
3. **Resolve** the initiator (§3). Fail closed — if resolution fails, ask the user before proceeding.
4. **Create** the folder on disk (with a Python `Path(...).mkdir(parents=True, exist_ok=True)` or equivalent). If a folder with the same slug already exists, ask the user before overwriting — this usually means the same slug was used twice or a previous session was interrupted.
5. **Create** the Linear ticket via `save_issue` with the template from §4. Capture the returned URL.
6. **Write** `brief.md` inside the folder (§5), with the Linear URL now known.
7. **Update** the Linear ticket description with the folder path — now both sides carry the link to the other.
8. **Confirm to the user** with a two-line status:
   > Workspace provisioned:
   > 📁 `content/{channel}/drafts/{slug}/`
   > 🎫 `{Linear ticket URL}` — assigned to {initiator name}
9. Proceed to Stage 3.

## §7 Idempotency

If the user re-invokes this skill on the same post (e.g. they closed Cowork and resumed later):

- **Folder exists + `brief.md` inside + `linear_ticket` field populated** → do not re-provision. Read `brief.md`, load the ticket, resume from the last unchecked stage in the workflow checklist.
- **Folder exists but no `brief.md`** → treat as a manual-created folder. Ask the user whether to adopt it (write `brief.md` into it, create Linear ticket) or use a different slug.
- **No folder yet** → run Stage 2.5 fresh.

The `brief.md` frontmatter is the source of truth for pair state. Don't rely on Linear ticket existence alone — tickets can be manually deleted or archived; `brief.md` reliably indicates whether provisioning has run for this slug.

## §8 Failure modes and what to do

| Symptom | Cause | Fix |
|---|---|---|
| Folder create fails with permission error | OneDrive sync issue or wrong path | Ask user to confirm path is writable; do not silently fall back to a different location |
| `save_issue` fails with 401 / auth | Linear MCP not authorized in this Cowork session | Surface error clearly: *"Linear MCP not authorized — connect it in Cowork settings and rerun."* Do NOT create folder without ticket, or vice versa |
| `list_users` returns multiple matches for same email | Duplicate Linear accounts | Ask user to pick one; cache the pick in memory |
| Slug collision with existing draft folder | Same title chosen twice, or interrupted previous session | Ask user: (a) resume that draft (§7 idempotency), (b) pick a new slug (suffix `-2`), or (c) archive the old one first |
| Linear project ID stale (project moved / renamed) | Project restructured since this doc was written | Re-run `list_projects team="Growth" query="Content"` to find current ID; update this doc's §2 table |
