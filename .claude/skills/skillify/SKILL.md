---
name: skillify
description: Extract a reusable, portable prompt blueprint from a source (gobbled note, pasted content, transcript, paper, workflow description) and save it to `07 potential skills/` as a standalone markdown file. The output is a generic, copy-pasteable template — not a working Claude Code skill. Use when the user says "skillify this", "extract the skill", "make this reusable", or asks for a prompt/workflow blueprint from any source.
---

# Skillify

Convert any operational pattern — a workflow, framework, prompt architecture, or repeatable process described in a source — into a portable, modular prompt blueprint stored in the vault's Skill Library.

The output is **not** an installed Claude Code skill. It's a markdown blueprint the user can copy/port into any project, any agent, any LLM. To promote a blueprint to a real Claude Code skill, that's a separate manual step.

## Inputs

- A vault note path (e.g., a recent gobble output)
- Pasted source content
- A URL or file path (in which case run gobble logic first to fetch)
- A verbal description of a workflow the user wants captured

If the source is unclear, ask.

## Steps

1. **Acquire the source.** Read the note / fetch the URL / take the paste.

2. **Identify the operational pattern.** What's actually being abstracted? Look for:
   - A repeatable workflow (steps in order)
   - A prompt architecture (role + context + instructions + format)
   - A framework (named structure with clear components)
   - An evaluation rubric or scorecard
   - A decision tree

   If the source has no such pattern, say so and stop. Don't fabricate.

3. **Abstract to a generic blueprint.** Strip source-specific details. What remains should work for any input that fits the pattern.

4. **Write the blueprint** to `07 potential skills/<kebab-case-name>.md` using this structure:

   ```markdown
   # <Title>

   **One-line purpose:** <what this skill does, in plain English>

   **When to use:** <triggers, situations, signals>

   **Source:** [[<original gobble note>]] or <URL> (captured YYYY-MM-DD)

   ## Inputs
   - <what the user provides>

   ## Prompt blueprint

   ```
   <The actual reusable prompt text, with {{placeholders}} for variables>
   ```

   ## Steps (if procedural)
   1. <step>
   2. <step>

   ## Output format
   <what the output should look like>

   ## Notes
   - <edge cases, gotchas, when NOT to use>
   ```

5. **Propose before writing.** Show the user the blueprint draft + intended filename. Wait for approval.

6. **Write.** Use the Write tool.

7. **Cross-link back to the source.** If the source is a vault note, append `[[<blueprint-name>]]` under its `## Related` section.

8. **Suggest promotion (optional).** If the blueprint is sharp enough to become a real Claude Code skill, mention it: "This could be promoted to `~/.claude/skills/<name>/SKILL.md` — want me to draft that next?"

## Naming

- kebab-case
- specific, not generic: `competitive-gap-matrix.md` not `analysis.md`
- include the noun: what kind of artifact does it produce?

## Don't

- Don't include source-specific names, brands, or examples in the blueprint body. Those go in the Notes section as illustration.
- Don't write the blueprint as prose — it's a template. Use placeholders, code blocks, clear sections.
- Don't skillify content that doesn't have a real reusable pattern. Some sources are just information; not everything becomes a skill.
- Don't duplicate an existing blueprint — check `07 potential skills/` first; if something similar exists, propose updating it instead.
