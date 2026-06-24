---
name: librarian-sweep
description: Weekly vault hygiene and cross-linking pass. Scans recently modified notes for missing frontmatter, missing Related links, MOC drift, inbox rot, naming violations, and stray files, then proposes a fix list and applies it on approval. Use when the user says /librarian-sweep, "sweep the vault", "link pass", or on a weekly schedule.
---

# Librarian sweep

Keep the vault a living library instead of a filing cabinet. Run weekly, or whenever asked. Read `09 vault operations/note standards.md` first → it defines the conventions this sweep enforces.

## Scope

Default: notes modified in the last 7 days (`find . -name '*.md' -mtime -7`, excluding `.claude/`, `.agents/`, `.obsidian/`, `.git/`, `99 archive/`). The user can widen ("sweep the last month") or narrow ("sweep 02 venture-one").

**Deliverable exemption (hard rule).** Skip client deliverables entirely → any note with `type: client-deliverable` OR under a `*/clients/*` path (image stacks, creative briefs, month content, mockups, one-pagers, QA banks). They are operational output, not knowledge: no link gate, no frontmatter requirement, no MOC check, never flagged as orphans. They stay reachable via their venture hub's dataview by folder. Exclude them from every check below so the sweep reports on the knowledge layer only, not factory exhaust. Filter them out up front:

```bash
find . -name '*.md' -mtime -7 \
  -not -path './.claude/*' -not -path './.git/*' -not -path './.obsidian/*' \
  -not -path './.agents/*' -not -path './99 archive/*' -not -path '*/clients/*' \
  | while read -r f; do grep -ql '^type: client-deliverable' "$f" || echo "$f"; done
```

## Checks

Run all eight, collect findings, fix nothing yet:

1. **Link gate.** Recently modified notes with fewer than 2 `[[wikilinks]]`, or no `## Related` section, or no link to their **domain hub** (`[[venture-one hub]]`, `[[knowledge hub]]`, etc.). For each, grep the vault for genuinely related notes (shared entities, clients, frameworks, topics). Propose links to satisfy the gate → one of them the domain hub → real semantic connections only, never manufactured. A note with no genuine relations gets flagged "no relations found, route to 00 inbox/", not a forced link.
2. **Missing frontmatter.** New notes (created during the scan window) missing the required `tags:` (1-4) and `type:` keys, or using tags/type outside the closed list in `10 resources/tag-vocabulary.md`. Old notes are exempt → no backfill.
3. **MOC drift.** Notes created during the window that aren't referenced from their domain hub (`venture-one hub`, `reading hub`, etc.). Propose the hub line to add.
4. **Inbox rot.** Anything sitting in `00 inbox/` → propose routing per the gobble heuristics.
5. **Naming violations.** New files/folders breaking lowercase convention (ASINs, `M500`, `CLAUDE.md`, `SKILL.md`, `README.md`, `AGENTS.md` exempt).
6. **Strays.** Files at the vault root that aren't `home.md`, `CLAUDE.md`, `AGENTS.md`, or dotfiles; new top-level folders beyond the canonical twelve.
7. **Stray binaries.** Binary files (PNG/JPG/PDF/video) that landed in the vault during the window → they belong in the assets store (`09 vault operations/assets store.md`). Exempt: images embedded in notes via `![[...]]`, `.excalidraw`, `10 resources/design-systems/`. Propose the mirrored `LifeOS Assets/` destination and the `## Assets` pointer to add.
8. **Skill-ref links.** `[[wikilinks]]` whose target matches a Claude Code skill name (`ls .claude/skills/`) → these dangle forever (skills aren't notes). Propose rewriting each to `` `skill-name` `` (backtick), or `[[skills-catalog|skill-name]]` if a clickable target is wanted. Self-healing: convert on approval. Skip occurrences inside `.claude/` and inside fenced code blocks.

## Report, then fix

Present one consolidated report:

```
LIBRARIAN SWEEP → <date>, window: <N> days, <M> notes scanned
1. LINKS    <note> → propose [[x]], [[domain hub]]  (reason, one line each)
2. FRONTMATTER  <note> → add tags: [<t>], type: <t>
3. MOC      <hub> → add <note> under <section>
4. INBOX    <file> → route to <destination>
5. NAMING   <path> → rename to <lowercase>
6. STRAYS   <path> → <action>
7. BINARIES <path> (<size>) → move to LifeOS Assets/<mirrored path>, add ## Assets pointer
8. SKILL-REF <note> → rewrite [[skill-name]] to `skill-name`
```

Binary moves only run on a Mac (iCloud Drive is unreachable from cloud sessions → report them as pending there).

Wait for approval (full or per-item). On approval:
- Add proposed links under `## Related` in *both* directions (the note and its targets).
- Insert frontmatter at the top of flagged new notes, preserving content untouched.
- Append MOC entries in the matching section, keeping the MOC's existing style.
- Route inbox items, apply renames with `git mv`, fix references to renamed paths.

Finish with one line per category: what was applied, what was skipped.

## Don't

- Don't touch note *content* beyond frontmatter insertion and `## Related` appends.
- Don't backfill frontmatter onto notes older than the scan window.
- Don't manufacture links to hit a quota → sparse-but-real beats dense-but-fake.
- Don't rename files that other notes or skills reference without updating the references in the same pass.
