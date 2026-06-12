---
name: opencode-operator-starter
description: Scaffold an operator's first terminal-agent setup that works on day one. Drops a business-agnostic AGENTS.md/CLAUDE.md-style context block (folder conventions, voice, do/don't) plus one runnable "weekly operator report" skill into OpenCode, Claude Code, Codex, or any agent. Provider-agnostic and copy-paste runnable, no subscription required. Use when an operator says "set up my first agent", "give me a starter config", "I just installed OpenCode", "make my agent useful day one", or wants the config-plus-first-skill starter pack instead of debating which coding agent to buy.
when_to_use: When an ecommerce operator, agency owner, or lean founder is wiring up their first terminal coding agent (OpenCode, Claude Code, Codex, or other) and needs a working context file plus one real business skill that runs immediately. Not for choosing an agent → for getting the one you picked doing useful work the same afternoon.
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
---

# OpenCode Operator Starter

A starter pack that turns a freshly installed terminal agent into a useful operator tool in one sitting. Two parts, both copy-paste runnable, both provider-agnostic:

1. **The context block** → a generic `AGENTS.md` (also works as `CLAUDE.md`) that teaches the agent your folder conventions, your voice, and your do/don't rules. This is the file every agent reads first. Get it right and every later prompt inherits it.
2. **The first skill** → a runnable "weekly operator report" the reader customizes to their store or agency. It reads a folder of business exports, summarizes the week in operator metrics, and writes a dated report. One command, one artifact.

The whole point: the decision was never "which agent." The decision is the config plus the first skill. Ship those and the agent stops being AI theater and starts saving a Friday afternoon.

## How it works

1. Confirm where the operator's agent reads its root context file. OpenCode and Codex read `AGENTS.md`; Claude Code reads `CLAUDE.md`. They are the same file with two names → write one, symlink or copy to the other.
2. Drop the context block (below) at the root of the operator's working folder. Fill the four `[BRACKETS]` → business name, what they sell, their folders, their voice.
3. Drop the weekly-report skill into the agent's skills/commands location (OpenCode: `.opencode/command/` or a `skills/` folder per the agent's docs; Claude Code: `.claude/skills/`; Codex: `.agents/skills/`). Keep the body identical → only the paths and metric list change.
4. Run it against a folder of exports. Read the output. Tune the metric list and the voice line. Re-run.

## The context block (AGENTS.md / CLAUDE.md)

Write this to the root of the operator's working folder. No private data, no machine-specific absolute paths → everything is relative and bracket-filled.

```markdown
# AGENTS.md → [BUSINESS NAME] operator context

You are the operating assistant for [BUSINESS NAME], a [WHAT YOU SELL / WHO YOU SERVE].
You help run the business leaner. You are not a chatbot → you read files, do the work, and write results back as markdown.

## Folder conventions

- `exports/`   raw data drops (CSV, screenshots, reports). Untriaged. Read-only → never edit files here.
- `reports/`   finished outputs you write. Dated when the doc is time-bound (e.g. `weekly-report-2026-06-10.md`).
- `reference/` standing context: SOPs, brand notes, product list, pricing. Read before you assume.
- `drafts/`    work in progress.

All filenames lowercase, dashes not spaces, `.md` extension. Dates only when the doc is date-bound.

## Voice

[ONE OR TWO LINES IN YOUR VOICE → e.g. "Direct and plain. Operator metrics over adjectives. No hype, no exclamation marks, no emojis. Write like you are briefing a busy partner who already knows the business."]

## Metrics that matter here

Frame everything in the numbers this business actually runs on:
[LIST YOURS → e.g. revenue, units, CVR, CTR, ACOS/TACOS, refund rate, ad spend, contribution margin, hours saved.]

## Do

- Read `reference/` before answering anything about how the business works.
- Show your inputs → name the files you used.
- Prefer a short table or a tight bullet list over a wall of prose.
- When a number is missing, write "(not in the data)" → never invent it.
- Write outputs to `reports/` or `drafts/`, not to chat.

## Don't

- Don't edit anything in `exports/`.
- Don't fabricate numbers, dates, or quotes.
- Don't add fluff, hype, or filler. If you have nothing to say, say less.
- Don't touch files outside this working folder.
```

## The first skill (weekly operator report)

Drop this where your agent loads skills/commands. The frontmatter shape matches Claude Code skills and reads cleanly as an OpenCode command or Codex skill → the body is plain instructions any agent follows.

```markdown
---
name: weekly-operator-report
description: Read this week's business exports and write a one-page operator report in [BUSINESS NAME]'s voice. Use when the operator says "run the weekly report", "what happened this week", or every Friday.
when_to_use: End of week, or any time the operator wants a fast read on the numbers from the latest exports.
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
---

# Weekly operator report

Produce a one-page weekly report from the files in `exports/`. Follow the AGENTS.md voice and do/don't rules.

## Steps

1. List every file in `exports/`. Note its date or coverage window if the filename or header says so.
2. Read each file. Pull the metrics that matter (see AGENTS.md "Metrics that matter here"). If a file is a screenshot, read the numbers off it.
3. Read `reference/` for any target, benchmark, or prior-week number to compare against.
4. Compute week-over-week change where you have both weeks. If you only have one week, say so → do not guess the prior week.
5. Write the report to `reports/weekly-report-YYYY-MM-DD.md` using today's date.

## Report shape

```
# Weekly report → [WEEK ENDING DATE]

## The number that matters
[The single most important metric this week, with direction vs last week.]

## What moved
[3-6 bullets. Each: metric, value, change, one-line why if the data shows it.]

## What needs attention
[1-3 things trending the wrong way or breaking. Operator framing, not alarm.]

## One decision for next week
[The single action the data argues for. Specific.]

## Sources
[List the exact files from exports/ you used.]
```

## Rules

- Operator metrics only. No adjectives where a number works.
- Missing data is "(not in the data)" → never invented.
- One page. If it runs long, cut, do not summarize into mush.
- Name your source files at the bottom. Every time.
```

## Make it yours

- Swap the metric list in both files for the numbers your business runs on. An Amazon seller leads with CVR, ACOS, sessions; an agency leads with retainer health, hours logged, deliverables shipped.
- Tighten the voice line to one sentence that sounds like you. The agent inherits it on every run.
- Add a second skill once the report runs clean → same shape, different job (a P&L read, a review-sentiment pass, a competitor scan).
- If your agent is OpenCode, the context file is `AGENTS.md` and commands live where its docs put them → the body here does not change, only the location.

## Gotchas

- **Two filenames, one file.** Claude Code wants `CLAUDE.md`, OpenCode/Codex want `AGENTS.md`. Write one, copy or symlink to the other → do not maintain two.
- **Empty `exports/` produces an empty report.** The skill is honest by design → if there is no data, it says so. Feed it real exports first.
- **Screenshots need a vision-capable model.** If the agent can't read numbers off an image, export the underlying CSV instead.
- **Relative paths only.** Keep everything inside the working folder → no machine-specific absolute paths, so the pack moves between machines and agents unchanged.

## What good looks like

The operator runs one command on Friday and gets a dated, one-page report that names its sources and ends with a single decision → instead of opening five dashboards and eyeballing it. The config and the first skill took an afternoon to wire and now run in under a minute. The agent earned its place before anyone paid for a subscription.
