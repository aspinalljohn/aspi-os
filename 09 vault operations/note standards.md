---
tags: [sop]
type: reference
---

# note standards

The minimal conventions every *new* note gets. The source of truth is the vault's `CLAUDE.md` and the closed [[tag-vocabulary]] → this doc is the operational expansion the weekly librarian-sweep enforces. No backfill → the standard grows forward through `/gobble`, `/skillify`, and the sweep. Don't mass-edit old notes to comply.

## frontmatter

Two keys required on every knowledge note, both drawn from [[tag-vocabulary]]:

```yaml
---
tags: [amazon/hero-image, content/linkedin]   # 1-4, closed list only
type: playbook                                 # exactly one, from the enum
---
```

- `tags` → 1-4, from the closed list in [[tag-vocabulary]]. Cross-cutting themes only, not every passing mention. Never invent a tag → if something genuinely recurs and isn't listed, add it to [[tag-vocabulary]] first, then use it.
- `type` → exactly one from the enum: `playbook · client-deliverable · reference · reflection · content-draft · research · sop`.

Optional keys, only when they carry weight:
- `created` → YYYY-MM-DD, for date-bound or lifecycle notes. The filename still stays date-free unless the doc itself is date-bound (daily/meeting notes).
- `venture` → `venture-one · venture-two · venture-three`, when the note lives in `02`–`04`.
- `status` → `active · done · archived`, only for things with a lifecycle (projects, client work). A note without `status` is just knowledge.

Exempt: functional files (`CLAUDE.md`, `AGENTS.md`, `README.md`) and published-blog notes with load-bearing frontmatter.

## linking (the link gate)

- At least **2 `[[wikilinks]]`** to existing notes, by real semantic connection → never manufactured. The links live in a `## Related` section at the foot of the note.
- One of those links points to the note's **domain hub** → `[[venture-one hub]]`, `[[venture-two hub]]`, `[[venture-three hub]]`, `[[knowledge hub]]`, `[[projects hub]]`, `[[me hub]]`, `[[resources hub]]`, `[[vault operations hub]]` → so the note is reachable from the map of content.
- The notes it links to get a backlink under their own `## Related`. Two-way links are what make the vault a brain instead of a filing cabinet.
- If you genuinely can't find 2 real connections, the note belongs in `00 inbox/` until it can be placed in context. A note linked to nothing is a note that doesn't exist.

## naming

Unchanged from `CLAUDE.md`: all lowercase, spaces allowed, no dates unless date-bound, `.md` extension. Uppercase only for canonical identifiers (ASINs, `M500`) and functional filenames (`CLAUDE.md`, `SKILL.md`, `README.md`, `AGENTS.md`).

## Related

- [[tag-vocabulary]] → the closed tag + type enum these standards draw from
- [[vault operations hub]] → the primitives that maintain the vault
