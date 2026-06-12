---
name: operator-capture
description: Turn a raw captured note (voice transcript, quick text, screenshot description) into structured, routed output → a one-line summary, a type classification (task / idea / decision / follow-up / content seed), the right destination/owner, a suggested next action, and any extracted dates. Built for operators capturing thoughts from their phone and routing them into their AI stack instead of losing them. Use when the user drops a raw note and wants it triaged, or says "capture this", "route this thought", "where does this go".
when_to_use: When an operator hands over a raw captured note (dictated voice memo, fast text, screenshot description) and wants it turned into a structured, routed item with a type, destination, owner, next action, and any dates. Provider-agnostic → runs in any chat assistant or coding agent.
allowed-tools:
  - Read
---

# Operator Capture

A thought you have on your phone is worth nothing until it lands somewhere with an owner and a next action. This skill takes one raw capture and returns a structured, routed item. No more lost voice memos. No more "I'll remember that."

You give it a raw note. It gives you back a triaged card you can act on or paste into your system.

## When to run it

- You dictated a voice memo walking between meetings and dropped the transcript here.
- You fired off a fast text note to yourself and want it sorted, not buried.
- You screenshotted something (a dashboard, a Slack message, an ad) and described what you saw.
- You have five half-thoughts from the day and want them each routed.

If it is a raw, unstructured thought from the field → run this.

## What it does

Takes one capture and returns exactly this card:

1. **Summary** → one line, plain English, what this actually is.
2. **Type** → one of: `task` / `idea` / `decision` / `follow-up` / `content seed`.
3. **Destination / owner** → where it goes and who acts on it.
4. **Next action** → the single concrete next step (a verb + an object).
5. **Dates** → any deadline, event, or time reference extracted and normalized.
6. **Confidence flag** → if the capture is ambiguous, it says so and asks one clarifying question instead of guessing.

## The classification rules

Pick the single best-fit type. Tie-break in this order: `decision` > `follow-up` > `task` > `content seed` > `idea`.

- **task** → there is a thing to do, an owner can do it, it has a finish line. ("reorder the B07 SKU before we stock out")
- **idea** → a possibility, not yet committed. No owner, no deadline. ("what if we ran the hero image as a video test")
- **decision** → a choice was made or needs to be made/recorded. ("we're killing the amber variant, CVR never recovered")
- **follow-up** → it depends on another person or a future moment. ("check with the 3PL on the Q4 cutoff after they reply")
- **content seed** → a thought worth turning into a post, email, or lesson later. ("the AI theater vs real leverage line landed in the call")

## The output format (always use this)

```
CAPTURE
-------
Summary:      <one line>
Type:         <task | idea | decision | follow-up | content seed>
Destination:  <system / list / person → where this goes>
Owner:        <who acts; "me" if the operator>
Next action:  <single concrete verb + object>
Dates:        <normalized date(s), or "none">
Priority:     <high | medium | low → from stakes + time pressure>
Note:         <only if the capture was ambiguous: one clarifying question>
```

If the note clearly contains **more than one** item, split it → return one CAPTURE card per item, numbered.

## How to run it (paste-and-go)

Paste this whole block into your assistant, then drop your raw note where marked.

```
You are my operator capture router. I will give you ONE raw note I captured
on my phone (a voice transcript, a fast text, or a screenshot description).

Turn it into a structured card using EXACTLY this format:

CAPTURE
-------
Summary:      <one line, plain English>
Type:         <task | idea | decision | follow-up | content seed>
Destination:  <where it goes → a list, tool, doc, or channel>
Owner:        <who acts; "me" if it's mine>
Next action:  <single concrete verb + object>
Dates:        <any deadline/event normalized to YYYY-MM-DD, or "none">
Priority:     <high | medium | low>
Note:         <only if ambiguous: ONE clarifying question>

Rules:
- Pick ONE type. Tie-break: decision > follow-up > task > content seed > idea.
- Next action must be a real next step, not a restatement of the note.
- Extract and normalize ALL dates. "next Friday", "EOM", "Q4 cutoff" → resolve
  to YYYY-MM-DD using today's date. If a date is relative and you don't know
  today, say so in Note.
- If the note has more than one item, return one numbered CAPTURE per item.
- Don't invent owners, dates, or destinations. If you have to guess, flag it
  in Note with one question.
- No preamble. Output the card(s) only.

My destinations (route to the closest match):
- Tasks → [your task manager / list]
- Decisions → [your decision log / notes]
- Content seeds → [your content backlog]
- Follow-ups → [your CRM / waiting-on list]
- Ideas → [your ideas parking lot]

Today is [YYYY-MM-DD].

Raw note:
"""
[PASTE YOUR CAPTURE HERE]
"""
```

## Worked example

**Raw note (voice transcript):**

> "ok note to self the willo listing the new hero image we tested it's actually beating control by a decent margin we should roll it to the other three SKUs but I want to check with marcus first he had a concern about the brand color also remind me the amazon brand registry call is thursday"

**Output:**

```
1. CAPTURE
-------
Summary:      New Willo hero image is beating control; consider rolling to 3 more SKUs.
Type:         decision
Destination:  Decision log → Willo creative
Owner:        me
Next action:  Confirm hero-image winner with Marcus before rollout (his brand-color concern).
Dates:        none
Priority:     high

2. CAPTURE
-------
Summary:      Amazon Brand Registry call is Thursday.
Type:         follow-up
Destination:  Calendar / waiting-on list
Owner:        me
Next action:  Hold time for the Brand Registry call.
Dates:        <Thursday → resolve to YYYY-MM-DD>
Priority:     medium
```

## Make it yours

- **Wire your real destinations.** Swap the placeholder destinations for your actual stack (Todoist, Notion, a Slack channel, a notes file). The router only routes as well as the map you give it.
- **Add an owner roster.** If you work with a team, list names + what they own so "Owner" fills with the right person instead of always "me".
- **Tune the types.** Add a type if your work needs one (e.g. `bug`, `client-reply`, `metric-check`). Keep the tie-break list updated.
- **Pin "today."** If you run this in an agent that doesn't know the date, hardcode today's date in the prompt so date extraction stays accurate.
- **Chain it.** Once the card is clean, hand it to your task-manager integration to actually create the task → the structured output is already in the right shape.

## Gotchas

- **Relative dates with no anchor.** "Next Friday" is meaningless without today's date. The prompt forces a Note flag when the date can't be resolved → fill in today before pasting.
- **Multi-item dumps read as one.** A rambling voice memo often hides three items. The split rule catches most; if it merges them, tell it "split into separate items".
- **Over-eager owners.** Don't let it assign work to a teammate it inferred. If Owner isn't explicit in the note, it should be "me" or a flagged question.
- **Content seed vs idea drift.** A content seed is publishable raw material; an idea is an unproven possibility. When unsure, it's an idea.
- **Don't let it write the essay.** A content seed is a one-line seed, not a finished post. Keep the output a card.

## What good looks like

- A captured thought becomes an actionable card in seconds, with an owner and a next step → zero lost notes.
- Dates land on the calendar instead of in your head.
- The card pastes straight into your task/decision system in the right shape (to confirm against your own stack).
- Time from "had a thought" to "it's routed" drops to near zero (to confirm with your own before/after).
