---
title: Tag Vocabulary
tags: [reference]
type: reference
---

# Tag Vocabulary

The **closed, controlled** tag set for this vault. These are the ONLY tags allowed → never invent freeform tags. Namespace them (`#family/tag`) so you can query a family (`#content`) or a specific theme (`#content/linkedin`). Tags capture **cross-cutting themes** that folders can't; they are not duplicates of folders.

Part of [[resources hub]] · [[vault operations hub]].

## Rules
- **Closed list only.** Apply tags from this page. If something genuinely recurs and isn't here, add it here *first*, then use it.
- **1-4 tags per note**, only where the note is substantially about the theme → not every passing mention.
- **One `type:`** per note from the enum below.
- Tags + type live in YAML frontmatter: `tags: [content/linkedin, ai/automation]` and `type: playbook`.

> The families below are **examples** → replace them with the cross-cutting themes of *your* work. Keep the structure (namespaced families + a single `type:` enum); swap the contents.

## Tags (example families → customize)

### `#content/*` → distribution & writing
- `content/linkedin` → LinkedIn posts, articles, carousels
- `content/newsletter` → newsletter / essays
- `content/video` → short-form video, scripts

### `#ai/*` → tooling & models
- `ai/agents` → coding agents, skills, hooks, config
- `ai/automation` → cron, scheduled/unattended jobs, MCP
- `ai/models` → model selection, pricing/routing

### `#ops/*` → running the business
- `ops/clients` → client delivery, onboarding
- `ops/sales` → pipeline, outbound, offers

### Standalone
- `sop` → standard operating procedures, runbooks
- `reflection` → personal thinking, journal, lessons learned

## `type:` enum (one per note)
- `playbook` → reusable strategy/framework/how-to
- `client-deliverable` → work produced for a specific client
- `reference` → specs, configs, cheat sheets, living reference
- `reflection` → personal thinking / journal
- `content-draft` → drafted post/article/essay
- `research` → digested external material, analysis, notes-on-source
- `sop` → step-by-step process doc
