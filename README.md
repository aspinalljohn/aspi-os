# Aspi OS

**A second brain that files itself.** This is the [Obsidian](https://obsidian.md) + [Claude Code](https://claude.com/claude-code) system I run my businesses on — packaged as a starter you can clone and make your own.

The idea is simple: you should never take notes by hand. You throw raw material at an AI steward — a link, a paste, a meeting transcript, a half-formed thought — and it structures it, connects it to everything related, and files it in the right place. Your vault stops being a filing cabinet and becomes a **linked graph that gets more valuable every time you add to it.**

This repo is the **method + a working starter**: the folder structure, the AI instructions that make Claude Code behave like a librarian, the rules that keep the graph connected, and ten portable skills you can run on day one. It ships with **zero of my actual notes** — it's a clean scaffold.

---

## The method in one screen

**1. Eleven numbered folders.** Lowercase, numbered for ordering. Your three daily ventures get first-class folders (`02`–`04`); everything else has a home.

```
00 inbox/            untriaged drop zone — stays near empty
01 reading/          digested articles, threads, research, reference
02 venture-one/      your first daily business
03 venture-two/      your second
04 venture-three/    your third
05 projects/         standalone, time-bound work
06 me/               personal: journal, health, reflection, drafts
07 potential skills/ portable prompt blueprints to port OUT
08 vault operations/ how the vault operates on itself
09 resources/        active reference kept at the ready
99 archive/          completed, dormant, retired
```

**2. An AI librarian, not a chatbot.** [`CLAUDE.md`](CLAUDE.md) turns Claude Code into a steward with hard rules. The two that matter most:

- **The link gate.** No note gets written without **at least 2 real `[[wikilinks]]`** to existing notes — one of them to the domain's hub. A note linked to nothing doesn't get created; it waits in the inbox. This is what keeps the graph from rotting into a pile of orphans.
- **Always propose a Routing Plan first.** Destination, filename, links, tags — you approve before anything is written.

**3. Maps of Content (hubs).** Every domain has a hub note at its folder root that indexes everything below it via [Dataview](https://github.com/blacksmithgu/obsidian-dataview). [`home.md`](home.md) is the front door — from it you reach any note in two clicks. It also surfaces orphan notes, cold notes to revisit, and a daily resurface pick.

**4. A closed tag vocabulary.** Notes get 1–4 tags and one `type:`, drawn only from a [controlled list](09%20resources/tag-vocabulary.md). No freeform tags, ever — so queries actually work.

**5. It files itself overnight.** A nightly job runs Claude Code headless to triage the inbox; a session-end hook backs the vault up to a private repo. See [automation patterns](09%20resources/automation-patterns.md).

---

## Quickstart

```bash
# 1. Use this template (GitHub) or clone it
git clone <this-repo> my-vault && cd my-vault

# 2. Open the folder as a vault in Obsidian. Install the Dataview community plugin.

# 3. Rename the placeholder ventures (02/03/04) to your real businesses,
#    and edit the venture hub notes to match.

# 4. Open the folder in Claude Code. It reads CLAUDE.md automatically.
#    Try it:
#       /gobble <paste a link or some raw text>
#    Claude reads your existing notes, summarizes the input, proposes a
#    routing plan with links + tags, and writes it on your approval.
```

That's it. The system works from the first `/gobble`.

---

## What's inside

| Path | What it is |
|---|---|
| `CLAUDE.md` | The librarian instructions — the heart of the system. `AGENTS.md` symlinks to it for Codex/other agents. |
| `home.md` | The front-door MOC with Dataview queries (recent, orphans, resurface, review queue). |
| `01–99` folders | The eleven-folder scaffold, each with its hub note. |
| `.claude/skills/` | Ten portable starter skills (below). |
| `.claude/settings.json` | SessionEnd hook template (backup + session log). |
| `08 vault operations/` | The vault primitives: `gobble`, `skillify`, note standards. |
| `09 resources/tag-vocabulary.md` | The closed tag set — customize to your domains. |
| `09 resources/automation-patterns.md` | How the nightly sweep, backup, and crons are wired. |
| `scripts/*.template` | Sanitized backup + inbox-sweep scripts. Edit paths, move outside the vault. |

### The ten starter skills

**Vault primitives**
- `gobble` — ingest any raw input → summarize → propose routing + links → write on approval
- `skillify` — extract a reusable prompt blueprint from any source
- `librarian-sweep` — weekly hygiene pass: fix missing links, frontmatter, MOC drift, inbox rot

**Operator skills** (business-agnostic, copy-paste runnable)
- `operator-capture` — turn a raw captured thought into a routed, typed item
- `overnight-ops` — define a scheduled unattended run that leaves a morning brief
- `automation-migration-audit` — find every call site of a deprecated CLI and plan the migration
- `mcp-ops-hub` — ask one question that spans several connected MCP tools, get one reconciled answer
- `volume-lane-router` — route each AI workflow to the cheapest model that should actually work
- `local-offload-router` — decide what can run on a self-hosted model vs. a hosted API
- `opencode-operator-starter` — scaffold a first terminal-agent setup that works day one

---

## Make it yours

1. **Rename the ventures.** `02/03/04` and their hubs → your real businesses. Search-replace `venture-one/two/three`.
2. **Add a "who you're working with" block** to the top of `CLAUDE.md` — your role, how you communicate, the lens you think in. This is what makes the AI sound like *your* steward.
3. **Rewrite the tag vocabulary** to your domains. Keep the structure (namespaced families + one `type:` enum); swap the contents.
4. **Wire the automations** when you're ready — copy the `scripts/*.template` files out to `~/scripts`, edit the paths, and point the backup at a **private** repo.

---

## Two rules that keep it safe

- **Your real vault is private.** Back it up to a *private* repo. This public scaffold is a *separate* repo with none of your content — never make your live vault public.
- **Secrets never enter the vault.** API keys live in `~/.env` or your keychain, referenced by scripts that live outside the vault.

---

## Why it works

Most note systems fail because filing is friction and orphan notes pile up until search is the only way in. Aspi OS removes the friction (the AI files for you) and forbids the orphans (the link gate). What you're left with is a knowledge base that compounds — every note makes the next one easier to connect, and the graph is always reachable from `home`.

Built by [Aspi](https://goaspi.com). MIT licensed — take it, fork it, make it yours.
