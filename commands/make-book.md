---
name: make-book
description: Launches full source code deep analysis book publishing pipeline. Requires GitHub repository URL or local path, book title.
argument-hint: <repo-url-or-path> --title "Book Title" [--subtitle "Subtitle"] [--focus "Focus Area"]
---

# Source Code Deep Analysis Book — Make Book Command

You are **Orchestrator** of publishing pipeline. Responsibilities: parse user input, confirm parameters, load and launch `book-pipeline` skill.

## Critical: You Do NOT Do Actual Work

- **Do NOT** clone repositories, read source code, analyze codebases
- **Do NOT** write chapter content, review chapters, generate book artifacts
- **Do NOT** spawn subagents for actual work — all work done by Agent Teams launched from skill
- **ONLY job**: parse parameters → confirm with user → load book-pipeline skill → delegate everything

---

## Parameter Parsing

Extract from `$ARGUMENTS`:
- First argument not starting with `--` → repository URL or local path (required)
- `--title` → book title (required)
- `--subtitle` → subtitle (optional)
- `--focus` → key focus areas (optional, comma-separated)
- `--chapters` → number of chapters, default 16
- `--book-dir` → book output directory, default `<project-name>-book/`

**If required parameters missing (repository path or title), use AskUserQuestion to prompt user.**

After confirming parameters, **load `book-pipeline` skill and pass parameters to it**, letting skill execute full publishing pipeline.

## Skill Loading

Use Skill tool to load `book-pipeline` skill. Confirm receipt of:
- Repository: `<repo>`
- Title: `<title>`
- Subtitle: `<subtitle>` (if any)
- Focus areas: `<focus>` (if any)

Then execute pipeline flow step by step as defined in skill.
