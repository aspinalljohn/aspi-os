---
name: gobble
description: Ingest raw input (URL, pasted text, transcript, file path) into the vault. Read across existing folders for related notes, summarize the input into structured markdown, propose a Routing Plan with destination + cross-links, then write on approval. Use whenever the user says "gobble this", "deep gobble", pastes a chunk of raw content, or hands you a link/PDF to file into the vault.
---

# Gobble

Turn raw external material into structured, cross-linked notes inside this Obsidian vault.

## Inputs

User provides one of:
- A URL → fetch it with WebFetch
- A file path (PDF, .md, .txt, .docx) → Read it
- Pasted raw text (X thread, transcript, article excerpt) → use directly
- A reference to a Fathom meeting → use Fathom MCP tools

If unclear, ask once what the source is.

## Steps

1. **Acquire content.** Pull the raw text via the appropriate tool.

2. **Scan the vault for related notes.** Grep filenames and content across `01 knowledge/`, `02 venture-one/`, `03 venture-two/`, `04 venture-three/`, `06 projects/`, `07 me/`, `08 potential skills/` for keywords from the input (key entities, frameworks, people, products). Identify 3–8 candidate notes for cross-linking. Don't search the entire vault → be targeted.

3. **Summarize the content.** Structured markdown, starting with YAML frontmatter:
   - `tags:` → 1-4 from the closed list in [[tag-vocabulary]] (`10 resources/tag-vocabulary.md`), only where the note is substantially on-theme. Closed list only → never invent tags.
   - `type:` → one value from the `type:` enum in [[tag-vocabulary]].
   - Then the body:
   - Title (Title Case, no clickbait)
   - **Source:** URL / file / "pasted" / Fathom recording link
   - **Date:** YYYY-MM-DD captured
   - **TL;DR:** 2–4 sentences
   - **Key points:** bulleted, the actual ideas → not formatting fluff
   - **Quotes / specifics:** any direct text worth preserving verbatim
   - **Related:** `[[Internal Links]]` to existing vault notes you identified in step 2 → **minimum 2 links, and one must be the domain hub** (`[[venture-one hub]]`, `[[venture-two hub]]`, `[[venture-three hub]]`, `[[me hub]]`, `[[knowledge hub]]`, `[[projects hub]]`, `[[resources hub]]`, `[[vault operations hub]]`). If you can't find 2 real connections, route to `00 inbox/` instead → a note linked to nothing doesn't belong in the library yet.

4. **Propose a Routing Plan.** Present to the user before writing:
   ```
   ROUTING PLAN
   Primary destination: <top-level folder>/<subfolder>/<Filename>.md
   Reasoning: <one line → why this destination, not inbox>
   Cross-links to update:
     - [[Existing Note 1]] → will add backlink in its Related section
     - [[Existing Note 2]] → will add backlink
   Confidence: <high | medium → drop in inbox if low>
   ```

5. **Wait for approval.** If the user says "go", "ship", "do it", proceed. If they propose changes, adjust. If silent, don't write.

6. **Write the file.** Use the Write tool. The note must satisfy the link gate (≥2 `[[links]]`, one being the domain hub). Then update any cross-linked existing notes by appending the new `[[link]]` under a `## Related` section (create the section if absent).

7. **Report.** One line: `Wrote <path>. Linked from N notes.`

## Routing heuristics

- **Digested information / article / thread / research / reference** → `01 knowledge/<Topic>/`
- **Venture one work** → `02 venture-one/<project>/`
- **Venture two work** → `03 venture-two/<project>/`
- **Venture three work** → `04 venture-three/<project>/`
- **Standalone project work (not one of the three ventures)** → `06 projects/<project>/`
- **Personal reflection / journal / health / peptides / linkedin draft** → `07 me/<subfolder>/`
- **Portable prompt blueprint detected** → suggest running `/skillify` (saves to `08 potential skills/`)
- **Unclear / multi-topic / triage later** → `00 inbox/`

## Don't

- Don't write before presenting the Routing Plan.
- Don't invent cross-links to notes that don't exist. Only link to files you've verified.
- Don't rewrite the source → preserve the original voice in quotes.
- Don't strip nuance for brevity. If the source is long and dense, the summary can be longer.
- Don't drop into `00 inbox/` as a default → only when routing confidence is genuinely low.
