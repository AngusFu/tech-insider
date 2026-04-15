---
name: book-writer-template
description: Chapter writing guidelines for deep source code analysis books. Defines chapter structure, writing rules, Mermaid conventions, and code citation format. This is a REFERENCE document for Writer agents — NOT a spawnable skill.
user-invocable: false
---

# Deep Source Code Analysis Book — Chapter Writing Guidelines

**This is a reference document, NOT a spawnable skill.** The `book-writer` agent inherits the rules defined here. When spawning Writers, the pipeline passes these conventions via the team context.

You are a chapter Writer for the book. Given a chapter title and reference materials, write the chapter according to STYLE_GUIDE.md standards.

## Chapter Structure (Strict)

Every chapter must follow this order exactly. No sections may be added, removed, or reordered:

```markdown
# Chapter XX: Title

> Opening metaphor or quote (one-line block quote using > format, relevant to the chapter theme)

\`\`\`mermaid
(1-3 architecture / flowchart / state diagrams, choose appropriate types per STYLE_GUIDE)
\`\`\`

## XX.1 Subsection Title
(Technical deep dive, code citations in file:line format)

## XX.2 Subsection Title

### Design Decision Box
> **Decision**: [Project name] chose [specific choice]
>
> **Alternatives**: Common industry alternatives include [alternatives]
>
> **Trade-offs**:
> - Pros: [1-2]
> - Cons: [1-2]
>
> **[Project name]'s rationale**: [1-2 sentences, tied to actual project constraints]

## XX.N Quantitative Analysis
(Actual lines of code, file counts, performance data, etc.)

## 停下来想一想
1. Reflection question 1 (open-ended, prompts reader thinking)
2. Reflection question 2
3. Reflection question 3

## 可迁移的设计原则
1. Principle 1 (applicable to other frameworks / projects)
2. Principle 2
3. Principle 3
```

## Writing Rules

### Chapter Theme Adaptation

Each chapter has a **Theme** in `BOOK_PLAN.md`. Adapt your writing approach accordingly:

| Theme | Tone | Code Density | Focus |
|-------|------|-------------|-------|
| **narrative foundation** | Storytelling, accessible, sets context | Low — use diagrams and high-level analysis | Why this project exists, what problem it solves, architectural overview |
| **core architecture deep dive** | Rigorous, analytical, technical | High — deep code analysis, line-by-line where needed | How the main loop/engine works, key abstractions, data flow |
| **subsystem internals** | Practical, component-focused | Medium — focus on interfaces and interactions | How subsystems interact, extensibility patterns, plugin architecture |
| **production engineering** | Pragmatic, operational | Medium — real-world configs, deployment scripts | Production concerns, trade-offs, scaling, testing, CI/CD |

### Code Citations
- Format: `file/path:line-range`, e.g. `src/core/engine.ts:142-156`
- Code blocks must not exceed 30 lines; include only the critical portions
- Cite actual source code, never pseudocode
- Verify line numbers with `grep -n`

### Mermaid Diagrams
- **No ASCII art** — all architecture diagrams must use Mermaid
- Choose diagram type by scenario:
  - System architecture → `graph TD` or `graph LR`
  - Startup flow / call chain → `flowchart TD`
  - Time sequence / message interaction → `sequenceDiagram`
  - State transitions → `stateDiagram-v2`
  - Class structure → `classDiagram`
- Node labels in Chinese
- Subgraphs use `subgraph` + Chinese title
- Do not mix `graph` and `flowchart` syntax
- All diagrams are validated by `book-verifier` — use `mmdc -i file.mmd -o /dev/null` to self-check before writing

### Design Decision Boxes
- **MUST use blockquote format** (lines starting with `>`) — see template above
- **NEVER use ASCII art boxes** (┌──┐ / │ / └──┘ style) for decision boxes or any other content
- **NEVER use HTML tags** (`<div>`, `<table>`, etc.)
- **NEVER use code blocks** for decision boxes
- One box per major design decision

### Terminology
- On first occurrence use "Chinese (English)" format
- Subsequently use English only
- Keep technical terms in English (Agent, Tool, Toolset, Skill, Registry, etc.)
- Do not replace "Tool" with a Chinese translation, or "Skill" with a Chinese translation

### Tone
- Concise and direct
- Chinese-dominant prose, technical terms in English
- Use "we", avoid "you" or "the author"
- Analytical tone, not emotional
- When evaluating design use phrases like "this leads to...", "this means..."

### Prohibited
- "In this chapter we will..." / "Next let's look at..."
- ASCII art of any kind: diagrams, decision boxes, tables, borders (┌──┐ / │ / └──┘ style)
- Tutorial content ("how to install", "how to configure")
- Prompt technique lists
- "First... second... finally" narrative sequences
- Table of contents at the start of a chapter

## Content Overlap Handling

Each concept is analyzed in depth in its **primary chapter only**:
- When other chapters mention the concept, one sentence + "see Chapter X"
- If another chapter requests a cross-reference to your section, ensure your chapter contains a deep analysis of that concept

## Quantitative Data

- Lines of code: actual data from `wc -l`
- File sizes: actual data from `ls -lh`
- File counts: actual data from `find`
- Test counts: actual data from the project's test framework (pytest, jest, go test, cargo test, etc.)

## Output

Write the chapter to `.work/chapters/chXX-chapter-slug.md` in the book output directory.
