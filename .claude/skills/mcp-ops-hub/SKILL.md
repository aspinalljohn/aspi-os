---
name: mcp-ops-hub
description: Route a natural-language operator question across a set of connected MCP tools (store, ads, email, calendar, tasks → kept generic), pull and reconcile the data from the right servers, and return one answer with the cross-system reasoning shown. Provider-agnostic and copy-paste runnable. Built for operators whose data is scattered across Seller Central, ad platforms, inbox, Slack, and a spreadsheet → instead of copy-pasting between tabs, you declare which MCP servers are available and what each is good for, then ask one question and let the agent fan out, reconcile, and answer. Use when an operator has two or more tools wired in over MCP and wants a single agent to answer questions that span them.
when_to_use: When an operator has connected two or more MCP servers (store/ads/email/calendar/tasks or similar) and wants to ask one plain-English question that needs data from several of them at once → "is the ad spend on the SKU that's about to stock out worth it?" → and get a single reconciled answer instead of opening five tabs. Provider-agnostic → runs in any agent that speaks MCP. For routing work to the cheapest model rather than the right tool, use [[volume-lane-router]].
allowed-tools:
  - Read
  - Write
  - Grep
  - Glob
  - AskUserQuestion
---

# MCP Ops Hub

One operator question in. One reconciled answer out. The agent decides which connected MCP servers to hit, pulls from each, reconciles the data across systems, and shows its work.

The default operator failure is not a dumb model → it's scattered data. Sales live in Seller Central, spend lives in the ad platform, the supplier email lives in the inbox, the launch date lives in the calendar, the follow-up lives in a task app. Answering "should I pause this campaign?" means opening five tabs and holding it all in your head. MCP fixes the wiring → this skill is the brain that sits on top of the wiring and answers the question.

This is **not** a tool that calls real APIs by itself. It reads a config of which MCP servers you have, what each is good for, and routes a question to the right ones. The actual data calls happen through whatever MCP tools the host agent exposes. Keep all server IDs, keys, and endpoints in your own config → never paste them into a skill or a prompt.

## Inputs

- **The question** (required): one plain-English operator question. Can span systems. If none given, ask → never invent it.
- **The tool registry** (required): the config block below, filled in with the operator's connected MCP servers. If the operator hasn't declared one, walk them through filling it before answering anything.
- **Freshness tolerance** (optional): how stale is acceptable per system (live / today / this week). Defaults to "today" and flags anything older.

## The tool registry (operator fills this in)

The operator declares which MCP servers are connected and what each is good for. This is the routing table → the agent reads it to decide where a question goes. Use placeholders only. Real IDs/keys live in the host agent's MCP config, never here.

```yaml
# mcp-ops-hub registry → declare your connected MCP servers.
# server_id is a LABEL the agent reasons over, not a secret. Real
# connection details (URLs, keys, OAuth) live in your agent's MCP
# config file, not in this registry.

registry:
  - server_id: store
    good_for: orders, revenue, units sold, inventory on hand, SKU status, buy box
    entities: [sku, asin, order_id]
    freshness: today            # how current the data usually is
    read_only: true

  - server_id: ads
    good_for: ad spend, ACOS, ROAS, impressions, clicks, campaign + keyword status
    entities: [campaign_id, sku, keyword]
    freshness: today
    read_only: true

  - server_id: email
    good_for: supplier threads, customer escalations, invoices, anything in the inbox
    entities: [thread_id, sender, subject]
    freshness: live
    read_only: true             # set false only if you let the agent draft/send

  - server_id: calendar
    good_for: launch dates, restock dates, promo windows, meetings
    entities: [event_id, date]
    freshness: live
    read_only: true

  - server_id: tasks
    good_for: open follow-ups, owners, due dates, what's in flight
    entities: [task_id, owner, due_date]
    freshness: live
    read_only: false            # the agent may create a follow-up task

# Join keys → how the agent reconciles the same thing across systems.
# Without these, "the SKU in store" and "the SKU in ads" stay strangers.
join_keys:
  sku: [store.sku, ads.sku]
  asin: [store.asin]
  date: [calendar.date, tasks.due_date]
```

## The routing procedure

Run this for every question. Show the work → the reconciliation is the value, not just the final line.

### 1. Parse the question into data needs
Break the plain-English question into the concrete facts it needs and which `good_for` bucket each fact lives in. Example → "Is the ad spend on the SKU about to stock out worth it?" needs: inventory on hand (store), days of cover (store), ad spend + ACOS for that SKU (ads), and any restock date (calendar).

### 2. Select the servers
Match each data need to a `server_id` via its `good_for`. Only select servers you actually need → do not fan out to all of them by reflex. If two servers could answer the same need, prefer the one whose `entities` match the question's subject.

### 3. Pull (read-only first)
Call the matching MCP tools the host agent exposes, scoped as tightly as possible (one SKU, one date range). Never write on a read question. Only a server marked `read_only: false` may be used for an action, and only when the question explicitly asks for one.

### 4. Reconcile across systems
Stitch the results together on the `join_keys`. The store's SKU and the ads' SKU are the same SKU → line them up. Resolve units (currency, timezone, date formats). If two systems disagree (store says 40 units, a spreadsheet says 60), surface the conflict instead of silently picking one.

### 5. Answer in one block
Return: the direct answer first, then the cross-system reasoning (which servers, what each said), then the freshness note (when each number was last current), then one recommended next action. If `tasks` is writable and the answer implies a follow-up, offer to create the task → do not create it unsolicited.

## Output shape

```
ANSWER → <one line, the decision the operator actually asked for>

REASONING (cross-system):
- store → <fact pulled, e.g. SKU B0XXXX: 38 units on hand, ~6 days cover>
- ads   → <fact pulled, e.g. campaign spend $/day, ACOS %>
- calendar → <fact pulled, e.g. restock ETA>
RECONCILED ON → <join key, e.g. sku=B0XXXX>

FRESHNESS → store (today), ads (today), calendar (live). Flag: <anything stale>

CONFLICTS → <none | system A says X, system B says Y → which I trusted and why>

NEXT ACTION → <one concrete step. If a task server is writable, offer to log it.>
```

## Worked example

**Question:** "Should I keep spending on the campaign for the moisturizer SKU?"

1. **Parse** → needs: that SKU's inventory + cover (store), that SKU's ad spend + ACOS (ads), restock date (calendar).
2. **Select** → store, ads, calendar. Not email, not tasks.
3. **Pull** → store: 38 units, ~6 days cover. ads: $42/day, ACOS 41%. calendar: restock ETA in 12 days.
4. **Reconcile** → join on `sku`. Cover (6 days) runs out before restock (12 days) → you'll stock out mid-campaign.
5. **Answer:**

```
ANSWER → Throttle the campaign now. You'll stock out in ~6 days but restock is 12 days out → full spend just buys clicks you can't fulfill.

REASONING (cross-system):
- store → SKU has 38 units, ~6 days of cover at current velocity
- ads   → campaign is $42/day at 41% ACOS (above your target)
- calendar → restock ETA is 12 days out
RECONCILED ON → sku

FRESHNESS → store (today), ads (today), calendar (live). No stale data.

CONFLICTS → none.

NEXT ACTION → Cut daily budget ~60% until restock lands, then ramp. Want me to log a "re-ramp campaign on restock date" follow-up in tasks?
```

## Gotchas

- **Auth and scopes.** A server in the registry that isn't actually authenticated will fail silently or half-answer. On a tool error, say so → never paper over a missing pull with a guess.
- **Tool sprawl.** Ten servers in the registry makes routing worse, not better. Keep it to the systems that answer real questions. Prune `good_for` lines that overlap.
- **Wrong-tool picks.** The most common failure is the agent reaching for the plausible server instead of the correct one. Tight, non-overlapping `good_for` and `entities` lines are what prevent this → vague descriptions cause misroutes.
- **Data freshness.** A "today" number read at 9am is stale by 5pm in a fast-moving account. Always emit the freshness note and flag anything past its tolerance → a confident answer on stale data is worse than no answer.
- **Write actions.** Default every server to `read_only: true`. Only flip `tasks` (or similar) to writable deliberately, and even then make the agent *offer* the action, not take it.

## Related
- [[volume-lane-router]] → route work to the cheapest correct model (this routes to the correct tool).
- [[opencode-operator-starter]] → stand up the agent this skill runs inside.
