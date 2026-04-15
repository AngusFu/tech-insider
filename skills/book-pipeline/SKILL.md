---
name: book-pipeline
description: Launch the deep source-code-analysis book publishing pipeline. Orchestrates Agent Teams working in parallel through the full flow from clone to final draft. Same as /tech-insider:make-book command.
user-invocable: true
---

# Deep Source-Code Analysis Book — Pipeline Orchestration

You execute the full book publishing pipeline. Move through phases sequentially, using **Agent Teams** for parallel work within each phase.

## Critical: You (Lead) Do NOT Do Actual Work

- **Do NOT** clone repositories, read source code, or analyze codebases yourself — delegate to teammates
- **Do NOT** write chapter content, review chapters, or generate any book artifacts
- **Do NOT** spawn subagents for actual work — all work is done by Agent Teams within each phase
- **Your ONLY job**: parse parameters → create teams → spawn teammates → monitor progress → consolidate results → report to user
- Exception: Phase 1 (Clone + Analyze), Phase 2 (Topic Selection), Phase 6d (Final Verdict), and Phase 11 (Delivery) are lead responsibilities because they are coordination/decision phases, not content-generation phases

## Agent Teams Rules

**This pipeline uses Agent Teams, NOT subagents.** Each parallel batch spawns independent teammates that share a task list and communicate directly.

### Core Constraints

1. **Team size MUST NOT exceed 6** — at any point, no more than 6 teammates are active simultaneously. This is a hard limit.
2. **Auto-shutdown on completion** — teammates that finish their assigned tasks MUST shut down and exit immediately. Do not leave idle teammates running.
3. **Reviewer pattern: fixed 4, serial processing** — review phases (6a/6b/6c/7) use exactly 4 teammates maximum regardless of chapter count. Each teammate processes multiple chapters serially via the shared task list: pick up next task → complete → pick up next → shut down when queue empty.
4. **Queue-based onboarding via config file** — when a non-review phase requires more than 6 workers, do NOT split into batches manually. The pipeline (lead) is responsible for queue management:
   - Write all pending teammate assignments to `.work/team-queue.md` with status (queued / onboarded / completed)
   - Spawn up to 6 teammates initially, mark them as "onboarded"
   - Monitor active teammates: when one completes and shuts down, mark it "completed" in the queue, then spawn the next "queued" teammate and mark it "onboarded"
   - Repeat until the queue is empty
   - Queue file format:
     ```markdown
     # Team Queue
     | # | Agent Type | Task | Status |
     |---|-----------|------|--------|
     | 1 | book-chapter-reviewer | Review ch01 | completed |
     | 2 | book-chapter-reviewer | Review ch02 | completed |
     | 7 | book-chapter-reviewer | Review ch07 | queued |
     ```
5. **One team per phase** — create a fresh team at the start of each parallel phase, clean up before moving to the next.
6. **Lead only coordinates** — the pipeline (lead) delegates work, monitors progress, and reports to the user. All reading, writing, reviewing, and proofreading is done by teammates.

---

## Parameter Parsing

Extract from user input:
- First argument not starting with `--` → repo URL or local path (required)
- `--title` → book title (optional; can also be AI-generated during topic-selection phase)
- `--subtitle` → subtitle (optional)
- `--audience` → target readers (optional, e.g. "Python developers", "Go backend engineers")
- `--focus` → key focus areas (optional, comma-separated)
- `--chapters` → suggested chapter count, default 16 (passed to Phase 2 topic selection as a hint)
- `--book-dir` → book output directory, defaults to `<project-name>-book/`

If the repo path is missing, use AskUserQuestion to prompt the user.

After determining `BOOK_DIR`:
```bash
# Check for existing book directory (re-entrancy check)
if [ -d "$BOOK_DIR/.work" ]; then
  echo "WARNING: Existing book directory found at $BOOK_DIR"
  echo "Previous intermediate files may be overwritten."
  # Ask user whether to clean up or continue
fi
mkdir -p "$BOOK_DIR" "$BOOK_DIR/.work"
```

---

## Pipeline Orchestration

Execute phases in order. After each phase completes, report to the user and confirm before proceeding.

### Phase 1: Clone + Analyze

```
Input: repo URL or local path
Output: codebase metrics (language distribution, file count, LOC, directory structure, test coverage)
```

1. If URL, `git clone`; if local path, verify it exists
2. **Detect language distribution** (polyglot):
   ```bash
   find . -type f \( -name "*.py" -o -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" -o -name "*.go" -o -name "*.rs" -o -name "*.java" -o -name "*.rb" -o -name "*.swift" -o -name "*.kt" -o -name "*.cs" \) | \
     sed 's/.*\.//' | sort | uniq -c | sort -rn | head -10
   ```
3. Gather metrics (replace `<ext>` with the detected primary language):
   ```bash
   find . -type f -name "*.<ext>" | wc -l
   find . -type f -name "*.<ext>" -exec cat {} + | wc -l
   find . -type d \( -name "test*" -o -name "spec*" -o -name "__test*" \) | wc -l
   find . -type d -maxdepth 3 | grep -v node_modules | grep -v .git | sort
   du -sh .
   ```
4. Read README.md and entry-point files to understand architecture
5. Present analysis results to the user; confirm before moving to topic selection

### Phase 2: Topic Selection

```
Input: codebase analysis results, user intent
Output: topic report (TOPIC.md)
```

This is the first decision point for publication. **Judge whether it's worth writing, for whom, and how — before doing anything.**

1. **Early exit check**: if the codebase is very small (< 1K LOC, single file, or trivial project), recommend to the user that this project may not warrant a full technical book. Offer to continue anyway if the user confirms.
2. Based on code analysis, generate topic report `TOPIC.md` containing:
   - **Project Positioning** — what the project is and what problem it solves
   - **Technical Highlights** — the 3-5 most noteworthy technical decisions or architectural designs
   - **Target Audience** — who will read this book and what prerequisites they need
   - **Writing Angle** — which lens to use (architecture analysis vs. usage guide vs. source-code walkthrough)
   - **Title Suggestions** — 3-5 candidate titles + subtitles
   - **Recommended Chapter Count** — 12-18 chapters (based on codebase complexity), respecting the `--chapters` hint if provided
   - **What Not to Cover** — topics unsuitable for deep analysis
3. Present the topic report to the user
3. After user confirmation or edits, proceed to outline

### Phase 3: Outline

```
Input: TOPIC.md + codebase
Output: BOOK_PLAN.md + STYLE_GUIDE.md + EDITORIAL_PLAN.md
```

1. Create a team named `outline-phase`
2. Spawn one **book-planner** teammate with `full-plan` mode, passing the codebase path, confirmed topic (TOPIC.md), `BOOK_DIR`, and the `--chapters` hint (if provided)
3. Assign task: generate `BOOK_PLAN.md`, `STYLE_GUIDE.md`, and `EDITORIAL_PLAN.md`
4. Teammate shuts down after producing all three files
5. Present outline summary to the user; confirm before moving to coordination

### Phase 4: Pre-Writing Coordination (Critical — Prevents Style Fragmentation)

```
Input: BOOK_PLAN.md + STYLE_GUIDE.md
Output: DEPENDENCIES.md (chapter dependency graph + cross-reference conventions)
```

**Before Writers start in parallel, make sure they all know where their boundaries are.**

1. Create team `coordination`
2. Spawn one **book-planner** teammate with `dependencies` mode, passing `BOOK_PLAN.md` and `STYLE_GUIDE.md`
3. Teammate reads both files and generates `DEPENDENCIES.md`:
   - "Home chapter" for each concept (who does the deep analysis)
   - Cross-reference conventions for other chapters (exact wording for "see Chapter X")
   - Content boundaries between Writers (who covers what, who doesn't)
   - Transition suggestions between adjacent chapters (how the end of one chapter naturally leads into the next)
4. Teammate writes `DEPENDENCIES.md` and shuts down

### Phase 4.5: Code Index (Token Budget Reduction)

```
Input: codebase + BOOK_PLAN.md
Output: CODE_INDEX.md (pre-computed code summary + call graph + architecture map)
```

**Writers and reviewers query this index instead of reading raw source — cuts token cost by 50%+.**

1. Create team `code-index`
2. Spawn one **book-planner** teammate with `code-index` mode, passing the codebase path and `BOOK_PLAN.md`
3. Teammate scans the codebase and produces `CODE_INDEX.md` containing:
   - **Module summary** — top-level directory → purpose → key files → file count → LOC
   - **Call graph** — entry points → core functions → leaf functions (top N most-referenced)
   - **Data flow map** — how data moves through the system (request → response lifecycle)
   - **Key constants / configs** — important thresholds, limits, defaults from code
   - **Test inventory** — test file locations, framework used, approximate coverage
   - **Architecture summary** — layer boundaries, import relationships, cross-cutting concerns
4. Teammate writes `CODE_INDEX.md` and shuts down
5. Writers use this index as their primary reference; drill into raw source only for specific citations
6. Technical reviewers cross-reference claims against this index first, then verify specific files

### Phase 5: First Draft Writing (Staged — Prevents Writer Drift)

```
Input: BOOK_PLAN.md + STYLE_GUIDE.md + DEPENDENCIES.md + CODE_INDEX.md
Output: chapter files `.work/chapters/chXX-*.md`
```

1. Create `.work/chapters/` directory for draft chapters
2. Read `BOOK_PLAN.md` to get all chapter assignments. Split chapters into 2 batches:
   - **Batch 1**: Foundation chapters (first 2-3 chapters that set the book's tone)
   - **Batch 2**: All remaining chapters
3. **Batch 1**: Create team `writing-batch1`, spawn one **book-writer** teammate with a detailed task containing:
   - Exact chapter titles, numbers, and descriptions from `BOOK_PLAN.md`
   - Key source files listed per chapter
   - Boundaries from `DEPENDENCIES.md` (what NOT to cover, cross-reference targets)
   - STYLE_GUIDE.md conventions (chapter structure, Mermaid rules, design decision box format)
   - Code INDEX.md as primary reference
   - Batch 1 writer's role: set the book's tone — writing must be exemplary in structure compliance
4. **Checkpoint**: After Batch 1 completes, review the output for style, depth, and tone alignment with STYLE_GUIDE.md. If acceptable, proceed to Batch 2.
5. **Batch 2**: Create team `writing-batch2`, split remaining chapters among up to 3 **book-writer** teammates. Each task must contain:
   - Exact chapter assignments (titles, numbers, descriptions from `BOOK_PLAN.md`)
   - Key source files per chapter
   - Boundaries from `DEPENDENCIES.md`
   - STYLE_GUIDE.md conventions
   - CODE_INDEX.md as primary reference
   - Path to Batch 1 output as style reference sample ("match the writing style and depth of this chapter")
6. All teammates shut down after writing their chapters
7. Collect results on completion; show progress to the user

### Phase 6: Three Reviews

#### 6a. Initial Review (Per-Chapter Structure Check)

```
Input: all chapter files `.work/chapters/ch*.md` + STYLE_GUIDE.md
Output: `.work/review-chXX.md` (one per chapter)
```

1. Create team `initial-review`
2. Create tasks for each chapter in the shared task list (one task per chapter)
3. Spawn 4 **book-chapter-reviewer** teammates (max 4 regardless of chapter count)
4. Each reviewer picks up the next unassigned task from the task list, reviews that chapter, writes `.work/review-chXX.md`, then picks up the next one — repeat until all tasks are done
5. Each reviewer shuts down immediately after all assigned tasks are completed

#### 6b. Technical Review (Fact-Checking — Critical!)

```
Input: all chapter files `.work/chapters/ch*.md` + source repo + CODE_INDEX.md
Output: `.work/tech-review-chXX.md` (one per chapter)
```

**This is not a format check — it is a fact check. Verify whether the code logic described in each chapter actually matches the source code.**

1. Create team `technical-review`
2. Create tasks for each chapter in the shared task list (one task per chapter)
3. Spawn 4 **book-technical-reviewer** teammates (max 4 regardless of chapter count)
4. Each reviewer picks up the next unassigned task from the task list, reviews that chapter against source code, writes `.work/tech-review-chXX.md`, then picks up the next one — repeat until all tasks are done
5. Each technical reviewer shuts down immediately after all assigned tasks are completed
5. Each reviewer checks:
   - **Code Logic** — does the claimed behavior match the actual code
   - **Architecture Description** — does the described architecture match the actual code structure
   - **Code Reference Context** — does the code at `file:line` actually implement what the chapter claims
   - **Data Accuracy** — are performance figures and numbers backed by evidence
   - **Design Decision Authenticity** — do the described "alternatives" and "trade-offs" actually exist
6. **Automated test verification** (if tests exist):
   - Detect test framework from `CODE_INDEX.md` test inventory
   - Run relevant tests to verify behavioral claims using the appropriate command for the detected framework:
     | Framework | Command |
     |-----------|---------|
     | pytest | `pytest -v tests/ --tb=short 2>&1 \| head -50` |
     | unittest (Python) | `python -m unittest discover -v 2>&1 \| head -50` |
     | jest | `npx jest --verbose 2>&1 \| head -50` |
     | vitest | `npx vitest run 2>&1 \| head -50` |
     | go test | `go test ./... -v 2>&1 \| head -50` |
     | cargo test | `cargo test 2>&1 \| head -50` |
     | JUnit/Maven | `mvn test -q 2>&1 \| head -50` |
     | Gradle | `./gradlew test 2>&1 \| head -50` |
     | Other | Fall back to manual source verification |
   - If a chapter claims "function X handles Y case correctly", verify a test exists that covers it
   - If the codebase has no tests, fall back to manual source verification

#### 6c. Cross-Chapter Review (Consistency)

```
Input: all `.work/chapters/ch*.md` + all `.work/review-chXX.md` + all `.work/tech-review-chXX.md` + STYLE_GUIDE.md
Output: `.work/review-consistency.md`
```

**To avoid context window explosion (18 chapters + all review reports = 100K+ tokens), split consistency review by Book Part:**

1. Create team `consistency-review`
2. Read `BOOK_PLAN.md` to identify Parts (typically 4-5 Parts)
3. Spawn up to 4 **book-consistency-reviewer** teammates (if fewer Parts than 4, spawn one per Part)
4. Assign each teammate one Part's scope. If a teammate finishes early and other Parts remain, assign the next unassigned Part
5. Each teammate reads only the chapters within their assigned Part + the global `STYLE_GUIDE.md` and `BOOK_PLAN.md`
6. Each teammate writes `.work/review-consistency-partN.md`
7. After all Part reviewers complete and shut down, **the pipeline lead consolidates**: read all `.work/review-consistency-partN.md` files and merge into a single `.work/review-consistency.md`

#### 6d. Final Review (Overall Quality — Done by Pipeline Agent)

```
Input: all review reports (`.work/`) + all tech-review reports (`.work/`) + all chapters (`.work/chapters/`) + `.work/review-consistency.md`
Output: final review verdict written to `.work/final-review-verdict.md`
```

The pipeline agent synthesizes all review results and makes the go/no-go decision:

1. Collect all initial review verdicts (`review-chXX.md`)
2. Collect all technical review verdicts (`tech-review-chXX.md`)
3. Read the cross-chapter consistency report (`review-consistency.md`)
4. Tally PASS/FAIL counts and identify critical issues
5. Write verdict to `.work/final-review-verdict.md`
6. Judge:
   - Any technical review FAIL → requires rework
   - Any initial review FAIL → requires rework
   - Consistency report FAIL → requires rework
   - All PASS → ready for publication → Phase 8

### Phase 7: Rework

```
Input: review reports (`.work/`) + tech-review reports (`.work/`) + original chapter files (`.work/chapters/`)
Output: revised `.work/chapters/ch*.md` files
```

1. Read `BOOK_PLAN.md` to determine which chapters have FAIL verdicts
2. Create team `rework-round-N` (N = 1 or 2)
3. **Rework is handled by up to 4 generic Writer teammates**, each serially processing their assigned FAIL chapters:
   - Create tasks for each FAIL chapter
   - Split tasks among up to 4 **book-writer** teammates (contiguous chapter ranges)
   - Each task must contain:
     - The specific chapter from `BOOK_PLAN.md` (title, description, key files)
     - All FAIL items from `review-chXX.md` (structural issues)
     - All FAIL items from `tech-review-chXX.md` (factual errors)
     - STYLE_GUIDE.md for reference formatting rules
   - Each Writer fixes structural issues + factual errors against actual source code, shuts down when done
4. **Factual errors first** — tech review "Wrong" items must be fixed first because they may affect cross-references in other chapters
5. After Writer teammates complete revisions and shut down, **re-run Phase 6a (initial review) + Phase 6b (technical review)** with new teams
6. Maximum 2 rework rounds; beyond that, report to the user for a decision

### Phase 8: Verification

```
Input: all chapter files `.work/chapters/ch*.md`
Output: `.work/verification-status.md`
```

1. Create team `verification`
2. Spawn one **book-verifier** teammate with task: run automated structure checks including Mermaid syntax validation
3. Before launching, check if `mmdc` is available:
   ```bash
   which mmdc >/dev/null 2>&1 && echo "mmdc available" || echo "mmdc not available — falling back to heuristic checks"
   ```
4. Mermaid validation:
   - If `mmdc` available: extract all ```mermaid blocks and validate with `mmdc -i block.mmd -o /dev/null` — authoritative check
   - If `mmdc` unavailable: run heuristic checks (unbalanced brackets, missing arrows, invalid directives)
5. Teammate writes `.work/verification-status.md` with Mermaid syntax verdict per diagram, then shuts down
6. Display verification results table. If any FAILs, list the issues, get user confirmation, then proceed to proofreading

### Phase 9: Three Proofreads + Preface (Parallel)

```
Input: all chapter files `.work/chapters/ch*.md` + STYLE_GUIDE.md + TOPIC.md + BOOK_PLAN.md
Output: `.work/proofread-1.md` + `.work/proofread-2.md` + `.work/proofread-3.md` + `preface.md`
```

1. Create team `proofread-preface`
2. Spawn 4 teammates with independent tasks (max 4 — within limit):
   - **book-proofreader** with `first-proofread` mode → writes `.work/proofread-1.md`
   - **book-proofreader** with `second-proofread` mode → writes `.work/proofread-2.md`
   - **book-proofreader** with `readability-pass` mode → writes `.work/proofread-3.md`
   - **book-preface-writer** → writes `preface.md`
3. Each teammate shuts down immediately after completing their work
4. Report to the user when all are complete

### Phase 9.5: Preface Review

```
Input: preface.md + TOPIC.md + BOOK_PLAN.md + STYLE_GUIDE.md
Output: `.work/preface-review.md`
```

The preface is the first thing readers see — review it before incorporation:

1. Create team `preface-review`
2. Spawn one **book-chapter-reviewer** teammate with task: review `preface.md` against TOPIC.md, BOOK_PLAN.md, and STYLE_GUIDE.md. **Use the preface-specific checklist below, NOT the chapter structural checklist**:
3. Teammate checks:
   - **Accuracy** — does the preface's project positioning match TOPIC.md?
   - **Tone** — matches STYLE_GUIDE conventions (analytical, "we", not tutorial)?
   - **Scope** — does it introduce without diving into technical details?
   - **Structure** — covers motivation, significance, target audience, how to use?
   - **Length** — 1-2 pages, no Mermaid, no code citations
   - **Factual claims** — any project metrics or claims that can be verified?
4. Teammate writes `.work/preface-review.md` and shuts down
5. If FAIL, fix the preface before Phase 10

### Phase 10: Synthesis (Chunked Processing)

```
Input: all `.work/chapters/ch*.md` + preface.md + `.work/final-review-verdict.md` + three review reports (`.work/`) + proofread reports (`.work/`) + `.work/preface-review.md` + STYLE_GUIDE.md + CODE_INDEX.md
Output: fixed chapters + fixed preface + 4 appendices + book-final.md
```

**To avoid context-window saturation, process in sequential passes. Each pass creates its own team.**

1. **Pass 1: P0 Fixes** — create team `synthesis-p0`, spawn one **book-editor-in-chief** teammate with `p0-fix` mode, assign task: decision-box formatting, ASCII art replacement, missing structure completion. Teammate shuts down after outputting fixed chapters
2. **Pass 2: P1 Fixes** — create team `synthesis-p1`, spawn one **book-editor-in-chief** teammate with `p1-fix` mode, assign task: content deduplication, cross-reference correction, data consistency. Teammate shuts down after outputting fixed chapters
3. **Pass 3: P2 Fixes** — create team `synthesis-p2-fixes`, spawn one **book-editor-in-chief** teammate with `p2-fixes` mode, assign task: terminology unification, difficulty buffering, transitions. Teammate shuts down after outputting fixed chapters
4. **Pass 4: Appendix Writing** — create team `appendices`, spawn one **book-editor-in-chief** teammate with `write-appendices` mode, assign task: read all fixed chapters, `CODE_INDEX.md`, and `STYLE_GUIDE.md`, then write 4 appendix files:
   - `appendix-A.md` (File Index): scan the codebase, list all source files by functional area
   - `appendix-B.md` (Tool Reference): extract from `CODE_INDEX.md` and chapter tool mentions
   - `appendix-C.md` (Design Decisions): extract all decision boxes from all chapters
   - `appendix-D.md` (Glossary): extract from `STYLE_GUIDE.md` glossary + terminology used in chapters
   Teammate shuts down after writing all 4 appendix files
5. **Pass 5: Final Compilation** — create team `compilation`, spawn one **book-editor-in-chief** teammate with `compile-final` mode, assign task: read all fixed chapters + all 4 appendices + preface, compile `book-final.md`. Teammate shuts down
6. Present synthesis results to the user (fix counts, final word count)

### Phase 10.5: Final Review (Quality Gate — Before Delivery)

```
Input: book-final.md
Output: `.work/final-review.md` (final verdict: PASS/FAIL)
```

**The last human-readable quality gate. Catches ASCII art residue, missing sections, broken cross-references, and Mermaid errors in the compiled manuscript.**

1. Create team `final-review`
2. Spawn one **book-final-reviewer** teammate with task: review the compiled `book-final.md` against the checklist below:
   - **ASCII art residue** — `grep -P '[│├└─┌┐┬┴┼]+'` — any match is FAIL
   - **Structural completeness** — every chapter has metaphor, Mermaid, "停下来想一想", "可迁移的设计原则"
   - **Decision box format** — `>` blockquote only, no ASCII boxes, no HTML
   - **Mermaid syntax** — validate all diagrams (mmdc or heuristic)
   - **Cross-reference integrity** — "see Chapter X" references point to existing chapters
   - **Overall quality** — word count anomalies, TODO markers, placeholder text
3. Teammate writes `.work/final-review.md` with PASS/FAIL verdict
4. If FAIL: list blockers, get user confirmation, fix before Phase 11
5. Teammate shuts down after writing the report

### Phase 11: Delivery

1. Show final draft statistics (`wc -l`, `wc -c`, chapter count)
2. Deliver final draft files to the user:
   - List all deliverable file paths: `book-final.md`, `preface.md`, `appendix-A.md` through `appendix-D.md`
   - Display a summary of what was produced and where to find each file
3. Display pipeline execution summary

---

## Failure Handling Principles

- **Topic Selection** → project unsuitable for a book (too small, too simple, insufficient docs); explain to the user
- **Outline** → analysis insufficient; request more information
- **Review FAIL** → rework, max 2 rounds
- **Verification FAIL** → list specific issues; continue after user confirmation
- **Synthesis** → record all P0/P1/P2 fixes; display in final report

## Progress Reporting

After each phase completes, output:
```
✅ Phase N: [Phase Name] — Complete
  - [Key Output 1]
  - [Key Output 2]
  - [Duration / File Count / Other Metrics]
```

When the entire pipeline finishes, output a full summary.
