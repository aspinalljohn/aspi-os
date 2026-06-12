---
name: volume-lane-router
description: Classify an operator's AI workflows by stakes (volume / day-to-day / high-stakes / long-horizon), assign each the cheapest model that should actually work, define the escalation trigger that bumps a job up a lane, and output a routing table plus a cost-per-deliverable note. Provider-agnostic, copy-paste runnable. Built for operators overpaying because they run bulk, low-stakes work on premium models out of habit. Use when you say /volume-lane-router, "route my volume work to the cheap lane", "which model for which workflow", "build me a model router", or hands over a list of AI workflows and wants the cheapest correct lane for each.
when_to_use: When an operator wants to stop paying premium-model rates for bulk, low-stakes work. Takes a list of their AI workflows and returns a routing table that sends volume jobs to the cheapest capable model and only escalates on a defined failure trigger. The cost-floor companion to model-escalation thinking → use this to route work DOWN to the cheap lane, not up to the frontier.
allowed-tools:
  - Read
  - Write
  - Grep
  - Glob
  - AskUserQuestion
---

# Volume Lane Router

Take a list of an operator's AI workflows and route each one to the cheapest model that should actually do the job. The default mistake is running bulk work (review tagging, classification, first-pass summaries, PDP variant drafts) on a premium model out of habit. The cheap-flash lane now exists and is good enough for volume. This skill builds the router.

The metric is **cost per deliverable**, not model prestige. A premium model that costs 10x and ships the same useful output as a cheap-flash model is just 10x more expensive. This skill finds the floor for each job and tells you the exact trigger that should bump a job up a lane.

Provider-agnostic by design. It reasons in **lanes**, not brand names → so it survives the next price cut and the next model launch. You map the lanes to whatever providers you run.

## The four lanes

Every workflow lands in one of four lanes. The lane decides the model class.

| Lane | What it is | Model class | Why |
|---|---|---|---|
| **volume** | Bulk, low-stakes, repeated, one-shot. Tagging, classification, first-pass summaries, PDP variant drafts, extraction, routing. | Cheap-flash lane (e.g. Gemini Flash-class ~$1.50/$9 per 1M, or a comparable cheap tier from any provider) | High count, low consequence per item. The cheap model is good enough and the volume is where the bill hides. |
| **day-to-day** | Normal operator work. Writing, research, SOPs, normal analysis, most coding, support replies that need judgment. | Mid lane (a solid default model, not the cheapest, not the frontier) | Needs reliability and decent judgment, but no long-horizon autonomy. The workhorse. |
| **high-stakes** | Bounded but consequential. Strategy memos, hard reviews, one artifact that has to be unusually sharp, a decision that moves real money. | Premium lane (the strong reasoning model) | Low count, high consequence. One better answer changes a real decision → the premium price is cheap against the downside. |
| **long-horizon** | The job keeps going. Multi-step agentic work, tool loops, deep audits across many inputs, internal tool builds, work that used to need constant human rescue. | Frontier lane (the most capable model) | The alternative to the frontier model here is not "same output for less" → it's a cheaper model getting halfway and you paying the difference in your own time. |

Default everything to the **lowest** lane that should work, then let the escalation trigger move it up only when it earns it.

## The classification rubric

For each workflow, score these and let the highest-scoring dimension set the lane.

1. **Count** → how many times does this run per day/week? High count pulls toward volume.
2. **Consequence per item** → if one item is wrong, what's the cost? High consequence pulls up out of volume.
3. **Context load** → does the answer depend on many files, transcripts, sheets, prior decisions? Heavy context pulls toward long-horizon.
4. **Tool/loop depth** → does it search, build, run, read failures, retry? Loops pull toward long-horizon.
5. **Ambiguity** → is the next step obvious, or does it need judgment? High ambiguity pulls up out of volume.
6. **Prior failure** → has a cheaper model already failed this and needed rescue? Real failure (not vibes) justifies a bump.

Rule of thumb: **start every workflow in the volume lane and force it to earn each promotion.** Most operator work that people run on premium models is volume work in a costume.

## The escalation trigger

A workflow does not live in one lane forever. Each routing-table row gets an **escalation trigger** → the specific, observable failure that bumps it up one lane. Not "if it feels off." A trigger you can actually watch:

- Volume → day-to-day: "if the cheap lane mislabels more than ~5% of items on a spot-check" (set your own threshold).
- Day-to-day → high-stakes: "if the output feeds a decision worth more than $X" or "if a human keeps rewriting more than half of it."
- High-stakes → long-horizon: "if the job needs more than ~3 tool calls / multiple files / autonomous follow-through to finish."
- Any lane → up: "if a cheaper model failed this exact job and needed human rescue more than twice."

Escalation is per-job and reversible. If the bigger model produces the same useful output as the cheaper one, downgrade it back.

## The cost-per-deliverable note

For each workflow, attach a one-line cost note. Do not compare model prices in a vacuum → compare cost per useful deliverable. Track:

- Model lane used.
- Rough input/output tokens per run (estimate is fine).
- Runs per period.
- Retries / human-rescue time.
- Whether the deliverable shipped.

A cheap lane that ships 95% of the time at a tenth of the price beats a premium lane that ships 100% → unless that 5% is high-consequence. The note makes the trade explicit instead of leaving it to habit.

## Process

1. **Get the workflow list.** If the operator hasn't pasted one, ask for it (use AskUserQuestion): "List your AI workflows → one per line, rough volume and what's at stake if each one is wrong." If a vault note already holds their stack, Read it first.
2. **Classify each workflow** against the rubric → assign a lane. Default down, force promotions to earn it.
3. **Assign the cheapest model class** that should work for that lane. Name it as a lane, not a locked brand → map to the provider the operator uses.
4. **Define the escalation trigger** per row → the observable failure that bumps it up.
5. **Write the cost-per-deliverable note** per row.
6. **Output the routing table** (markdown) + a short summary: how many workflows moved to the cheap lane, the rough monthly saving directionally (mark any number "(estimate)" unless the operator gave real figures → never fabricate).
7. **Flag the watch-list** → the 2-3 workflows most likely to need escalation, so the operator knows where to spot-check first.

## Output format

A markdown routing table:

```markdown
| Workflow | Lane | Model class | Escalation trigger | Cost-per-deliverable note |
|---|---|---|---|---|
| Review tagging (bulk) | volume | cheap-flash | mislabel >5% on spot-check → day-to-day | ~500 runs/wk, tiny tokens → pennies; premium here is pure waste |
| Customer email replies | day-to-day | mid | human rewrites >50% → escalate | judgment needed, moderate volume; mid lane is the floor |
| Monthly margin audit | high-stakes | premium | needs >3 files / tool loop → long-horizon | 1 run/mo, moves real money → premium is cheap vs a wrong call |
| Internal tool build | long-horizon | frontier | n/a (already top lane) | rescue cost is hidden in your time → frontier pays for itself |
```

Then a 3-4 line summary: count moved to cheap lane, directional monthly saving "(estimate)", and the watch-list.

## Guardrails

- **Provider-agnostic.** Reason in lanes. Only name a specific model when the operator names their provider, and even then mark prices "(to confirm)" → they move.
- **No fabricated savings.** Only quote a dollar saving if the operator gave real volume/price figures. Otherwise mark "(estimate)" or describe the direction, not a number.
- **Default down, not up.** The whole point is the cost floor. The sibling move (escalating up to the biggest model) is a different decision → this skill is the down-route.
- **No private or client data, no machine-specific paths** in any output meant to ship.
- **Escalation is reversible.** Always tell the operator they can downgrade a job if the bigger model didn't change the outcome.
