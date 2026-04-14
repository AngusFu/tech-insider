---
name: book-pipeline
description: Launch the deep source-code-analysis book publishing pipeline. Orchestrates Agent Teams working in parallel through the full flow from clone to final draft. Same as /tech-insider:make-book command.
user-invocable: true
---

# Deep Source-Code Analysis Book — Pipeline Orchestration

You execute the full book publishing pipeline. Move through phases sequentially, using **subagent parallelism within each phase** as needed.

---

## Parameter Parsing

Extract from user input:
- First argument not starting with `--` → repo URL or local path (required)
- `--title` → book title (optional; can also be AI-generated during topic-selection phase)
- `--subtitle` → subtitle (optional)
- `--audience` → target readers (optional, e.g. "Python developers", "Go backend engineers")
- `--focus` → key focus areas (optional, comma-separated)
- `--book-dir` → book output directory, defaults to `<project-name>-book/`

If the repo path is missing, use AskUserQuestion to prompt the user.

After determining `BOOK_DIR`:
```bash
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

1. Based on code analysis, generate topic report `TOPIC.md` containing:
   - **Project Positioning** — what the project is and what problem it solves
   - **Technical Highlights** — the 3-5 most noteworthy technical decisions or architectural designs
   - **Target Audience** — who will read this book and what prerequisites they need
   - **Writing Angle** — which lens to use (architecture analysis vs. usage guide vs. source-code walkthrough)
   - **Title Suggestions** — 3-5 candidate titles + subtitles
   - **Recommended Chapter Count** — 12-18 chapters (based on codebase complexity)
   - **What Not to Cover** — topics unsuitable for deep analysis
2. Present the topic report to the user
3. After user confirmation or edits, proceed to outline

### Phase 3: Outline

```
Input: TOPIC.md + codebase
Output: BOOK_PLAN.md + STYLE_GUIDE.md + EDITORIAL_PLAN.md
```

1. Launch the **book-planner** agent, passing the codebase path, confirmed topic (TOPIC.md), and `BOOK_DIR`
2. Generate:
   - `BOOK_PLAN.md` — detailed chapter outline
   - `STYLE_GUIDE.md` — writing style guide
   - `EDITORIAL_PLAN.md` — editorial pipeline plan
3. Present outline summary to the user; confirm before moving to coordination

### Phase 4: Pre-Writing Coordination (Critical — Prevents Style Fragmentation)

```
Input: BOOK_PLAN.md + STYLE_GUIDE.md
Output: DEPENDENCIES.md (chapter dependency graph + cross-reference conventions)
```

**Before 5 Writers start in parallel, make sure they all know where their boundaries are.**

1. Generate chapter dependency graph `DEPENDENCIES.md`:
   - "Home chapter" for each concept (who does the deep analysis)
   - Cross-reference conventions for other chapters (exact wording for "see Chapter X")
   - Content boundaries between Writers (who covers what, who doesn't)
   - Transition suggestions between adjacent chapters (how the end of one chapter naturally leads into the next)
2. Send `DEPENDENCIES.md` to each Writer as a pre-writing reference
3. Confirm all Writers are aware of boundaries before starting drafts

### Phase 4.5: Code Index (Token Budget Reduction)

```
Input: codebase + BOOK_PLAN.md
Output: CODE_INDEX.md (pre-computed code summary + call graph + architecture map)
```

**Writers and reviewers query this index instead of reading raw source — cuts token cost by 50%+.**

1. Scan the codebase to produce `CODE_INDEX.md` containing:
   - **Module summary** — top-level directory → purpose → key files → file count → LOC
   - **Call graph** — entry points → core functions → leaf functions (top N most-referenced)
   - **Data flow map** — how data moves through the system (request → response lifecycle)
   - **Key constants / configs** — important thresholds, limits, defaults from code
   - **Test inventory** — test file locations, framework used, approximate coverage
   - **Architecture summary** — layer boundaries, import relationships, cross-cutting concerns
2. Writers use this index as their primary reference; drill into raw source only for specific citations
3. Technical reviewers cross-reference claims against this index first, then verify specific files

### Phase 5: First Draft Writing (Staged — Prevents Writer Drift)

```
Input: BOOK_PLAN.md + STYLE_GUIDE.md + DEPENDENCIES.md + CODE_INDEX.md
Output: chapter files chXX-*.md
```

1. Assign chapters from `BOOK_PLAN.md` to Writers:
   - **book-writer-foundation** → Foundation chapters (first 2-3 chapters)
   - **book-writer-core-loop** → Core loop chapter
   - **book-writer-core-system** → Core systems chapter
   - **book-writer-tools** → Tools / subsystems chapter
   - **book-writer-integration** → Integration and engineering chapter
   > Each Writer reads their assigned chapters from `BOOK_PLAN.md`, boundaries from `DEPENDENCIES.md`, and code summaries from `CODE_INDEX.md`
2. **Batch 1**: Launch **book-writer-foundation** first (1-2 chapters). These set the tone for the entire book.
3. **Checkpoint**: After Batch 1 completes, review the output for style, depth, and tone alignment with STYLE_GUIDE.md. If acceptable, proceed to Batch 2.
4. **Batch 2**: Launch remaining **4 Writers in parallel**, using the Batch 1 output as a style reference sample.
5. Collect results on completion; show progress to the user

### Phase 6: Three Reviews

#### 6a. Initial Review (Per-Chapter Structure Check)

```
Input: all chapter files ch*.md + STYLE_GUIDE.md
Output: `.work/review-chXX.md` (one per chapter)
```

1. Launch a **book-chapter-reviewer** agent for each ch*.md file (per-chapter structure check)
2. **Launch multiple reviewers in parallel**
3. Output `.work/review-chXX.md` to `BOOK_DIR/.work/`

#### 6b. Technical Review (Fact-Checking — Critical!)

```
Input: all chapter files ch*.md + source repo + CODE_INDEX.md
Output: `.work/tech-review-chXX.md` (one per chapter)
```

**This is not a format check — it is a fact check. Verify whether the code logic described in each chapter actually matches the source code.**

1. Launch a **book-technical-reviewer** agent for each ch*.md file
2. **Launch multiple technical reviewers in parallel**
3. Each reviewer checks:
   - **Code Logic** — does the claimed behavior match the actual code
   - **Architecture Description** — does the described architecture match the actual code structure
   - **Code Reference Context** — does the code at `file:line` actually implement what the chapter claims
   - **Data Accuracy** — are performance figures and numbers backed by evidence
   - **Design Decision Authenticity** — do the described "alternatives" and "trade-offs" actually exist
4. **Automated test verification** (if tests exist):
   - Detect test framework from `CODE_INDEX.md` test inventory
   - Run relevant tests to verify behavioral claims:
     ```bash
     # Example: if pytest tests exist
     pytest -v tests/ --tb=short 2>&1 | head -50
     ```
   - If a chapter claims "function X handles Y case correctly", verify a test exists that covers it
   - If the codebase has no tests, fall back to manual source verification
5. Output `.work/tech-review-chXX.md` to `BOOK_DIR/.work/`

#### 6c. Cross-Chapter Review (Consistency)

```
Input: all ch*.md + all `.work/review-chXX.md` + all `.work/tech-review-chXX.md` + STYLE_GUIDE.md
Output: `.work/review-consistency.md`
```

1. Launch **book-consistency-reviewer** skill to check cross-chapter consistency
2. Check: do different chapters contradict each other on the same technical point
3. Output `.work/review-consistency.md`

#### 6d. Final Review (Overall Quality — Done by Pipeline Agent)

```
Input: all review reports (`.work/`) + all tech-review reports (`.work/`) + all chapters + `.work/review-consistency.md`
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
Input: review reports (`.work/`) + tech-review reports (`.work/`) + original chapter files
Output: revised ch*.md files
```

1. For each FAIL chapter, send rework instructions to the corresponding Writer (including structural issues + factual errors)
2. **Factual errors first** — tech review "Wrong" items must be fixed first because they may affect cross-references in other chapters
3. After Writer revisions, **re-run Phase 6a (initial review) + Phase 6b (technical review)**
4. Maximum 2 rework rounds; beyond that, report to the user for a decision

### Phase 8: Verification

```
Input: all chapter files ch*.md
Output: `.work/verification-status.md`
```

1. Launch **book-verifier** agent to run automated structure checks including Mermaid syntax validation
2. Mermaid validation:
   - Extract all ```mermaid blocks from each chapter
   - Try rendering with `mmdc` (mermaid CLI) if installed — authoritative check
   - If `mmdc` unavailable, run heuristic checks (unbalanced brackets, missing arrows, invalid directives)
3. Output `.work/verification-status.md` with Mermaid syntax verdict per diagram
4. Display verification results table. If any FAILs, list the issues, get user confirmation, then proceed to proofreading

### Phase 9: Three Proofreads + Preface (Parallel)

```
Input: all chapter files ch*.md + STYLE_GUIDE.md + TOPIC.md + BOOK_PLAN.md
Output: `.work/proofread-1.md` + `.work/proofread-2.md` + `.work/proofread-3.md` + `preface.md`
```

Launch simultaneously (no inter-dependencies):
- **First Proofread** (text proofreading) → use **book-proofreader** skill's first-proofread mode → `.work/proofread-1.md`
- **Second Proofread** (cross-references) → use **book-proofreader** skill's second-proofread mode → `.work/proofread-2.md`
- **Third Proofread** (readability + narrative coherence + tone consistency) → use **book-proofreader** skill's readability pass mode → `.work/proofread-3.md`
- **Preface** → launch **book-preface-writer** agent → `preface.md`

Report to the user when all are complete.

### Phase 9.5: Preface Review

```
Input: preface.md + TOPIC.md + BOOK_PLAN.md + STYLE_GUIDE.md
Output: `.work/preface-review.md`
```

The preface is the first thing readers see — review it before incorporation:

1. **Accuracy** — does the preface's project positioning match TOPIC.md?
2. **Tone** — matches STYLE_GUIDE conventions (analytical, "we", not tutorial)?
3. **Scope** — does it introduce without diving into technical details?
4. **Structure** — covers motivation, significance, target audience, how to use?
5. **Length** — 1-2 pages, no Mermaid, no code citations
6. **Factual claims** — any project metrics or claims that can be verified?

Write findings to `.work/preface-review.md`. If FAIL, fix before Phase 10.

### Phase 10: Synthesis (Chunked Processing)

```
Input: all ch*.md + preface.md + three review reports (`.work/`) + proofread reports (`.work/`) + `.work/preface-review.md` + STYLE_GUIDE.md
Output: fixed chapters + fixed preface + book-final.md
```

**To avoid context-window saturation, process in three passes:**

1. **Pass 1: P0 Fixes** — launch **book-editor-in-chief** agent to handle all P0 issues (decision-box formatting, ASCII diagram replacement, missing structure completion), output fixed chapters
2. **Pass 2: P1 Fixes** — launch a new **book-editor-in-chief** agent to handle all P1 issues (content deduplication, cross-reference correction, data consistency), output fixed chapters
3. **Pass 3: P2 Fixes + Final Compilation** — launch a new **book-editor-in-chief** agent to handle P2 issues (terminology unification, difficulty buffering, narrative transitions), write 4 appendices, compile final draft `book-final.md`
4. Present synthesis results to the user (fix counts, final word count)

### Phase 11: Delivery

1. Show final draft statistics (`wc -l`, `wc -c`, chapter count)
2. Deliver final draft files to the user
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
