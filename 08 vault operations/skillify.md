---
tags: [ai/claude-code]
type: reference
---

# Skillify

**Primitive:** extract a portable, reusable prompt blueprint from any source → save to `07 potential skills/`.

Where [[gobble]] *captures* knowledge, skillify *abstracts a repeatable pattern* out of it. Point it at a gobbled note, a transcript, a paper, or a described workflow and say *"skillify this."* It:

1. Reads the source.
2. Identifies the operational pattern (workflow, prompt architecture, framework, rubric, decision tree). If there's no real reusable pattern, it stops — not everything becomes a skill.
3. Abstracts to a generic blueprint with `{{placeholders}}`, stripping source-specific detail.
4. Proposes the blueprint + filename, waits for approval.
5. Writes to `07 potential skills/<kebab-case-name>.md` and cross-links back to the source.

## Output ≠ executable skill

The result is a **copy-pasteable template** — portable to any project, agent, or LLM. It is *not* a runnable Claude Code skill. Promoting a blueprint to a real skill (`.claude/skills/<name>/SKILL.md`) is a separate, deliberate step.

Runnable source: `.claude/skills/skillify/SKILL.md`. See also [[gobble]], [[grill me skill]].
