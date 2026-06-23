---
title: Automation Patterns
tags: [reference, ai/automation]
type: reference
---

# Automation Patterns

The automations that make this vault run itself. These are **patterns**, not turnkey configs → every one needs your paths and your accounts. Part of [[vault operations hub]] · [[resources hub]].

> Golden rule: **the live scripts live OUTSIDE the vault** (e.g. `~/scripts/`), never inside it. On macOS that keeps Full Disk Access from resetting after OS updates, and it keeps the scripts from being committed into the vault. The vault only holds the `.template` copies and this doc.

## 1. Backup on every session end + hourly

The vault is a git repo pushed to a **private** remote. Two triggers keep it backed up:

- **SessionEnd hook** (`.claude/settings.json`) → runs `vault-backup.sh` (commit + push) every time a Claude Code session ends in the vault.
- **Hourly launchd/cron** → runs the same script to catch changes made outside Claude Code (Obsidian edits, mobile sync).

Template: [[vault-backup.sh.template|scripts/vault-backup.sh.template]]. Never point this at a public repo → your notes are private. The public/shareable artifact is a *separate* scaffold repo (this one).

## 2. Nightly inbox sweep

A launchd agent (macOS) or cron job (Linux) fires `inbox-sweep.sh` at ~23:30. It runs Claude Code **headless** (`claude -p "<prompt>"`) against the vault to format every item dropped in `00 inbox/`, cross-link it under the link gate, route it to the right folder, and write a single triage report. Fast-exits with no noise when the inbox is empty.

Template: `scripts/inbox-sweep.sh.template`.

## 3. Session log on session end

The second SessionEnd hook writes a short AI-authored daily log of what the session did to `07 me/session logs/YYYY-MM-DD.md`. Watch for two gotchas: **recursion** (a session-log write can itself trigger another SessionEnd → guard against it) and **inner-settings** (a headless `claude -p` call inherits the same hook, so it tries to log its own run → give the inner call a stripped settings file).

## 4. Headless content / report crons

Any recurring "do work while I sleep" job is the same shape: a launchd/cron entry → a shell wrapper that sets `HOME`/`PATH` explicitly (schedulers start with a minimal env) → a `claude -p "<prompt>"` call with `--permission-mode acceptEdits` → output appended to a log. The [[overnight-ops]] and [[automation-migration-audit]] skills generalize this.

## launchd vs cron

- **macOS** → `launchd` plist in `~/Library/LaunchAgents/`. Reload with `launchctl unload/load`. Keep plists and scripts in `~/scripts`, not the vault.
- **Linux** → `crontab -e`. Same wrapper-script approach.

## Checklist before you wire anything up

- [ ] Private git remote created and `git push -u origin main` done once by hand
- [ ] Scripts copied out of the vault to `~/scripts/`, paths edited, `chmod +x`
- [ ] `claude` CLI on `PATH` inside the wrapper (schedulers don't inherit your shell)
- [ ] Secrets in `~/.env` or the keychain → **never** committed, never in the vault
- [ ] Inner headless calls given a stripped settings file to avoid hook recursion
