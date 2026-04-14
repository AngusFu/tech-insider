---
name: make-book
description: Launches the full source code deep analysis book publishing pipeline. Requires a GitHub repository URL or local path, and a book title.
argument-hint: <repo-url-or-path> --title "Book Title" [--subtitle "Subtitle"] [--focus "Focus Area"]
---

# Source Code Deep Analysis Book — Make Book Command

You are the **Orchestrator** of the entire publishing pipeline. Your responsibilities: parse user input, confirm parameters, load and launch the `book-pipeline` skill to execute the full pipeline. **All actual work is done by the skill; you only handle parameter parsing and launch.**

---

## Parameter Parsing

Extract from `$ARGUMENTS`:
- First argument not starting with `--` → repository URL or local path (required)
- `--title` → book title (required)
- `--subtitle` → subtitle (optional)
- `--focus` → key focus areas (optional, comma-separated)
- `--chapters` → number of chapters, default 16
- `--book-dir` → book output directory, default `<project-name>-book/`

**If required parameters are missing (repository path or title), use AskUserQuestion to prompt the user to provide them.**

After confirming parameters, **load the `book-pipeline` skill and pass the parameters to it**, letting the skill execute the full publishing pipeline.

## Skill Loading

Use the Skill tool to load the `book-pipeline` skill. Confirm receipt of:
- Repository: `<repo>`
- Title: `<title>`
- Subtitle: `<subtitle>` (if any)
- Focus areas: `<focus>` (if any)

Then execute the pipeline flow step by step as defined in the skill.
