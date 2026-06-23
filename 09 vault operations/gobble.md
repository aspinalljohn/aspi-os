---
tags: [ai/claude-code, ai/automation]
type: reference
---

# Gobble

**Primitive:** ingest raw external material → structured, cross-linked note in the vault.

Throw Claude anything raw → a URL, a pasted X thread, a PDF, a transcript, a Fathom call → and say *"gobble this."* It:

1. Acquires the content (WebFetch / Read / paste / Fathom MCP).
2. Scans `01 knowledge/`, `02 venture-one/`, `03 venture-two/`, `04 venture-three/`, `06 projects/`, `07 me/`, `08 potential skills/` for related notes.
3. Summarizes into structured markdown (TL;DR, key points, quotes, related links).
4. Proposes a **Routing Plan** → destination + cross-links → and waits for approval.
5. Writes the note and back-links it from the related notes it found.

## Routing targets

| Input type | Lands in |
|------------|----------|
| Article, thread, research, reference | `01 knowledge/` |
| Venture one work | `02 venture-one/` |
| Venture two work | `03 venture-two/` |
| Venture three work | `04 venture-three/` |
| Standalone project (not a venture) | `06 projects/` |
| Personal reflection, health, drafts | `07 me/` |
| Reusable prompt pattern | → run `/skillify` (`08 potential skills/`) |
| Genuinely unclear | `00 inbox/` |

Runnable source: `.claude/skills/gobble/SKILL.md`. See also [[skillify]].
