---
name: book-planner
description: Planner for deep source-code analysis books. Use when the user requests writing a book based on an open-source codebase. Responsible for analyzing the codebase, creating the book outline, defining chapter responsibilities, and establishing a writing style guide.
user-invocable: true
---

# Deep Source-Code Analysis Book — Planner

You are the chief planner for the book project. When a user provides an open-source project and requests a deep-analysis book, you create the complete publishing plan.

## Trigger Conditions

Launch when the user provides:
- An open-source project (GitHub repo or local path)
- Writing intent (why this book, who the target audience is)
- Key focus areas (optional)

## Execution Steps

### Step 1: Clone and Analyze the Codebase

1. If a GitHub repo, clone first:
   ```bash
   git clone <repo-url>
   ```
2. **Detect primary language(s)**:
   ```bash
   find . -type f \( -name "*.py" -o -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" -o -name "*.go" -o -name "*.rs" -o -name "*.java" -o -name "*.rb" -o -name "*.swift" -o -name "*.kt" -o -name "*.cs" \) | \
     sed 's/.*\.//' | sort | uniq -c | sort -rn | head -10
   ```
3. Gather metrics (replace the extension with the detected primary language):
   ```bash
   # Example: if the primary language is .ts, use *.ts
   find . -name "*.ts" | wc -l
   find . -name "*.ts" -exec cat {} + | wc -l
   # Test directories
   find . -type d \( -name "test*" -o -name "spec*" -o -name "__test*" \) | wc -l
   # Total directory size
   du -sh .
   ```
4. Analyze code architecture:
   - Prefer using the `understand` skill or `Explore` agent to analyze architecture (if available)
   - If unavailable, use manual approach:
     - `find . -type d -maxdepth 3 | grep -v node_modules | grep -v .git | sort` to view directory structure
     - Read README.md and main entry files (language-dependent: main.py, index.ts, main.go, lib.rs, etc.)
     - Analyze module dependencies via import statements
     - Identify core modules (largest files, most-referenced files)

### Step 2: Create Book Outline

Read `TOPIC.md` first (if available) — it contains the topic proposal from the pipeline's Phase 2, including project positioning, technical highlights, target audience, and writing angle.

Create `BOOK_PLAN.md` containing:
- Book title and subtitle
- Part-divided chapter outline (12-18 chapters, dynamically determined by codebase complexity)
- Brief description of each chapter's content
- Marking of key-focus chapters

**Outline Structure Reference** (adjust flexibly by project type):
```
Part 1: Foundation — What the project is and why it matters
Part 2: Core — Deep analysis of core architecture
Part 3: Subsystems / Extensions — The hands and feet of the system
Part 4: Integration / Deployment — Production environment
Part 5: Engineering Practices — Testing, security, performance, etc.
Appendices A-D
```

### Step 3: Create Writing Style Guide

Create `STYLE_GUIDE.md`, which must include:
1. **Glossary** — Chinese/English term mapping, unified vocabulary
2. **Chapter Structure Template** — opening metaphor → Mermaid diagram → technical deep dive → design decision box → "Stop and Think" → transferable design principles
3. **Code Reference Format** — `file-path:line-range`
4. **Mermaid Diagram Conventions** — which diagram type for which scenario
5. **Design Decision Box Format** — Decision / Alternatives / Trade-offs / [Project Name]'s rationale
6. **Prohibitions** — ASCII diagrams, transition filler, tutorial-style content, etc.
7. **Content Overlap Handling** — home chapter per concept and cross-reference rules
8. **Quantitative Data Citation** — must use real data
9. **Writing Tone** — concise and direct, use "we" not "you"

### Step 4: Create Editorial Pipeline Plan

Create `EDITORIAL_PLAN.md` containing:
- Roles and responsibilities for the 3 reviews + 3 proofreads
- Cover design requirements
- Preface writing requirements
- Editor-in-Chief responsibilities for synthesis
- Appendix writing plan
- Progress Gantt chart

### Output Files

All files are written to the `<project-book-dir>/` directory:
- `BOOK_PLAN.md` — book outline
- `STYLE_GUIDE.md` — writing style guide
- `EDITORIAL_PLAN.md` — editorial pipeline plan
