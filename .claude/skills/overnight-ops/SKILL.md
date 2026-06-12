---
name: overnight-ops
description: Define a scheduled, unattended operator run that gathers business data while you sleep and leaves a decision-ready morning brief in your inbox or Slack. Pulls from your sources (sales, ads, inventory, inbox, alerts), checks each against thresholds and anomaly rules, drafts a prioritized brief, and delivers it on a cron schedule. Built failure-safe so a missing source never produces a silent empty no-op. Provider-agnostic and copy-paste runnable on any terminal agent (Claude Code, Codex, OpenCode) or any cron-capable host. Use when an operator says "set up an overnight agent", "wake up to a brief", "schedule a morning report", "automate my daily monitoring", or wants their freshest morning hours back instead of spending them assembling reports by hand.
when_to_use: When an ecommerce operator, agency owner, or lean founder wastes their best morning hours assembling reports and scanning for problems a scheduled agent could have prepared overnight. Turns a manual every-morning monitoring-and-reporting habit into one unattended run that gathers, checks, flags, drafts, and delivers a decision-ready brief before they wake. For routing which model runs the job cheaply, pair with [[volume-lane-router]]; for the loop-vs-hooks distinction, see [[loop vs hooks - 2026-06-09]].
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
---

# Overnight Ops

A scheduled, unattended operator run. It executes while you sleep, gathers your business data, checks it against rules you set, drafts a prioritized morning brief, and delivers it → so you wake up to a decision, not a chore.

This is not a chatbot you talk to. It is a **run** you set once and forget. The win is your first two morning hours back: instead of opening five dashboards and assembling the same report by hand, you open one brief that already knows what changed and what needs you.

Provider-agnostic. Runs on any terminal agent (Claude Code, Codex, OpenCode) or any cron-capable host. No machine-specific paths, no client data baked in → you fill the brackets with your own.

## What it does

1. **Pull** → read every source you list: sales, ads, inventory, inbox, alerts. Each source is just a file, an export, or a command that returns text.
2. **Check** → run each pulled value against your thresholds (ACOS over X, inventory under Y days, refunds spiking) and flag anomalies.
3. **Draft** → write a morning brief: top-line numbers, what's flagged, and a prioritized action list (do-first at the top).
4. **Deliver** → send the brief by email or Slack, timestamped, so it's waiting when you wake.
5. **Fail loud** → if any source is unreachable, the brief says so explicitly. It never silently ships a clean-looking report built on missing data.

## The run definition (copy-paste)

Save this as `overnight-ops.md` (or paste it as a skill / command in your agent). Fill every `[BRACKET]` with your own values. Nothing here is machine-specific.

```markdown
# Overnight Ops run

You are an unattended operator agent. It is overnight. No human is watching.
Produce a morning brief from the sources below and deliver it. Do not ask
questions → if something is ambiguous, make the safe call and note it in the brief.

## 1. Sources to pull
Read each source. Each is a file path, an export, or a command that returns text.
If a source is unreachable, do NOT skip silently → record it under "Source failures"
and keep going with the rest.

- SALES:     [path to today's sales export OR command, e.g. ./exports/sales-latest.csv]
- ADS:       [path to ad report export OR command]
- INVENTORY: [path to inventory export OR command]
- INBOX:     [path to a dump of unread/flagged customer messages OR command]
- ALERTS:    [path to any monitoring/error log OR command]

## 2. Checks to run (flag anything that trips)
- Sales:     flag if revenue is down more than [20]% vs the trailing [7]-day average.
- Ads:       flag any campaign with ACOS above [35]% OR spend up more than [50]% day-over-day.
- Inventory: flag any SKU with less than [14] days of cover at current velocity.
- Inbox:     flag any message containing [refund, broken, defective, chargeback, urgent].
- Alerts:    flag any line containing [error, failed, 5xx, down].
- Anomalies: flag any number that is more than [2]x or less than [0.5]x its trailing average,
             even if no explicit threshold above caught it.

## 3. Brief to draft
Write a short, scannable brief. No fluff. Operator metrics, not adjectives.

### Morning brief → [DATE]
**Top line:** revenue, ad spend, ACOS, units → today vs trailing [7]-day avg.
**Flagged (do first):** every tripped check, most costly at the top, one line each,
  with the number that tripped it and the obvious next action.
**Quiet:** one line confirming the sources that came back clean (so silence is verified, not assumed).
**Source failures:** any source that was unreachable, named explicitly. If this section
  is non-empty, put "⚠ INCOMPLETE BRIEF" at the very top of the message.

## 4. Deliver
Send the brief to [email address OR Slack channel/webhook]. Subject/line:
"Morning brief → [DATE]" (prefix "⚠ INCOMPLETE → " if there were source failures).

## 5. Failure-safe rule (non-negotiable)
- Never deliver a brief that LOOKS clean when a source failed. A missing source is a
  flagged event, not a no-op.
- If ZERO sources could be read, send a one-line alert: "Overnight run could not read any
  source → check the pipeline." Do not send an empty brief.
- Cap your own work: stop after [one] full pass. Do not retry a failing source more than
  [twice] or loop → an unattended run must not burn budget overnight chasing a dead source.
```

## Install and run

1. **Drop the file.** Save the run definition above as `overnight-ops.md` in your project (or register it as a skill/command in your agent so you can call it by name).
2. **Wire your sources.** Replace each `[BRACKET]` in section 1 with a real path or command. Start with the two sources that matter most → you can add the rest later. A source can be as simple as a nightly CSV export your store already produces.
3. **Set your thresholds.** Edit section 2 to your numbers. If you don't know them yet, leave the defaults and tune after the first week of briefs.
4. **Pick a delivery target.** Email is simplest (most agents and hosts can send mail or hit a webhook). Slack incoming webhooks work the same way → drop the webhook URL in section 4.
5. **Set the schedule (the part everyone forgets).** This run is only "overnight" if something triggers it overnight. Pick one:
   - **Native scheduler in your agent** → if your tool supports scheduled background tasks (e.g. Google Antigravity 2.0's scheduled tasks, or your agent's job scheduler), register the run there and set it for ~[5:00 AM] local. This is the cleanest option → the agent wakes itself.
   - **System cron** → on any always-on host, add one line. Example for 5 AM daily:
     ```
     0 5 * * * cd /path/to/project && your-agent run overnight-ops.md >> overnight-ops.log 2>&1
     ```
   - **Hosted scheduler** → a CI cron, a serverless scheduled function, or a workflow runner triggering the same command on a timer.
   The host must be on at run time → schedule it on an always-on machine or a cloud runner, not your laptop that's asleep too.
6. **Test it awake first.** Run it once manually at noon and read the brief. Confirm every source resolves and the failure-safe section behaves (temporarily point one source at a missing file and check the brief flags it). Only then trust it overnight.

## Make it yours

- **Add a source** → one more line in section 1 plus its check in section 2. Returns shipping, review velocity, a competitor price scrape, your cash position → anything that returns text.
- **Change the cadence** → weekly Monday brief instead of daily? Same file, change the cron to `0 5 * * 1`.
- **Tier the delivery** → quiet days get a short brief; any flagged item also pings your phone. Add: "If anything is flagged, also send a one-line SMS/push."
- **Route the model** → run the gather-and-check on a cheap fast model and reserve a stronger model only when something is flagged and the brief needs judgment. See [[volume-lane-router]].

## Gotchas (what breaks at 5 AM with nobody watching)

- **Silent no-op** → the worst failure. A source path changes, the pull returns empty, and you get a clean brief that's a lie. The failure-safe rule in section 5 exists for exactly this → the brief must announce missing sources loudly, never absorb them.
- **Stale data** → the run fires before your nightly exports finish writing. The brief reports yesterday's yesterday. Fix → schedule the run after your exports land, and have the brief print each source's timestamp so stale data is visible.
- **Runaway cost** → an unattended agent with no cap can retry a dead source, loop, or spawn subagents all night. Section 5 caps the pass and retry count for a reason. Set a hard budget/time limit on the run if your host supports it.
- **Dead host** → the schedule is on a machine that was asleep or offline. Nothing runs and you don't notice for days. Fix → run on an always-on host, and have the brief's absence be its own alert (if you don't get a brief by [6 AM], something's wrong).
- **Alert fatigue** → thresholds set too tight flag everything, so you stop reading. Tune after a week → a brief you skim is worse than no brief.

## What good looks like

You wake up. One message is waiting. It opens with the three numbers you'd have checked anyway, then a short list of what actually needs you, hardest-hitting first, each with the number that tripped it. On a clean night it's four lines and you move on. On a bad night, the thing that would've cost you is already at the top before you've had coffee.

You did not open five dashboards. You did not assemble anything. The run did the gathering and the checking; you do the deciding. That's the trade → unattended labor overnight, human judgment in the morning. (Time-saved and catch-rate numbers depend on your sources and thresholds → measure your own over the first two weeks. (to confirm))

## Related
- [[loop vs hooks - 2026-06-09]] → the loop-vs-hooks distinction this run sits on top of.
- [[volume-lane-router]] → route the cheap gather work to a cheap model, escalate only the judgment.
- [[opencode-operator-starter]] → the first-skill starter this is a natural second step after.
