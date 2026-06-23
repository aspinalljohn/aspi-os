---
title: Decision Log
tags: [ai/automation, reference]
type: reference
---

# Decision Log

The vault's **tool-agnostic memory**. One place for the standing decisions, conventions, and gotchas any AI agent working in this vault should know before it acts. Read it at the start of a session; propose an addition whenever a durable decision gets made.

Why this exists: Claude Code keeps its own auto-memory, but other agents (Codex, etc.) can't see it. This file is the shared mirror → it keeps every agent on the same standing rules without re-deriving them each session. Part of [[resources hub]] · [[vault operations hub]].

## How to use it

- **At session start:** skim this file for rules relevant to the task.
- **When a durable decision is made:** propose a one-line entry under the right section (the owner approves, same as the link gate).
- **Keep entries atomic and dated** where it matters. Convert relative dates ("next month") to absolute ones.
- **Don't put secrets here.** API keys live in `~/.env` or the keychain, never in the vault.

---

## Standing rules

> How the owner wants the agent to behave. Corrections, confirmed approaches, hard preferences.

- _(example)_ Never invent tags → closed vocabulary only. See [[tag-vocabulary]].
- _(example)_ Propose a Routing Plan before writing any note.

## Identity & accounts

> Which account/identity to use for which service, so the agent never guesses.

- _(example)_ Git identity for this vault: `name / email`.
- _(example)_ Publishing handle / primary email / sender address.

## Automations

> Scheduled or unattended jobs that touch the vault, and how they're wired.

- _(example)_ Nightly inbox sweep → see [[automation-patterns]].
- _(example)_ Session-end backup → private repo.

## Gotchas

> Things that have bitten before. Read these before debugging.

- _(example)_ Always run from the vault root (paths are vault-root-relative).

## Venture / domain facts

> Durable facts about the businesses and projects that don't live in any single note.

- _(example)_ Venture one positioning, key constraints, who's who.
