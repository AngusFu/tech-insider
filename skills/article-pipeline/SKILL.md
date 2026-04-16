---
name: article-pipeline
description: Launch the article writing pipeline. Orchestrates Agent Teams for web research, outline, writing, review, and delivery. Supports repo URL / reference URLs / idea modes.
user-invocable: false
---

# Article Pipeline — Orchestration

You execute the article writing pipeline. Move through phases sequentially, using **Agent Teams** for parallel work.

**Critical: You (Lead) Do NOT Do Actual Work**
- Do NOT clone repos, fetch URLs, or write content — delegate to teammates
- Do NOT write article sections — delegate to writers
- Your job: coordinate, delegate, report

## Lead's Operating Loop

```
Make decision (create tasks, spawn teammates)
  ↓
Wait for mailbox (teammate completion / idle / user response)
  ↓
Make next decision (shutdown, next phase, fix, deliver)
  ↓
Wait for mailbox
  ↓
(repeat)
```

**CRITICAL: After SendMessage to any teammate, yield immediately.** Do NOT make more tool calls.

---

## Parameter Parsing

Extract from user input:
- First non-`--` argument → repo URL / reference URL / topic keyword
- `--title` → article title (required)
- `--topic` → specific topic/focus (optional, overrides first arg for idea mode)
- `--urls` → comma-separated reference URLs (optional)
- `--word-count` → target word count (default 5000, range 3000-10000)
- `--audience` → target readers (optional)
- `--focus` → key focus areas (optional)
- `--article-dir` → output directory (default `<topic>-article/`)

### Input Mode Detection

| First Arg | Mode | Phase 1 Action |
|-----------|------|----------------|
| `https://github.com/*` | **Repo Mode** | Clone + analyze source code |
| `https://*` (non-GitHub) | **URL Mode** | Treat as reference URL, use web-fetch |
| Text/keywords | **Idea Mode** | Use web-search to gather context |

---

## Team Lifecycle

**ONE team throughout**: Create `article-pipeline` at Phase 2, delete at Phase 7.

### Shutdown Pattern (every phase end)
1. Send `shutdown_request` to each teammate (structured message format)
2. Yield immediately — wait for idle confirmations
3. Create tasks for next phase
4. Spawn all new teammates in parallel
5. Yield — wait for completion

---

## Pipeline Flow

### Phase 1: Input Analysis

```
Input: repo URL / reference URLs / topic keyword
Output: Mode detection + initial context
```

1. Detect input mode from first argument
2. **Repo Mode**:
   - Run `git clone` (if not exists)
   - Gather metrics (language, LOC, structure)
3. **URL Mode**:
   - Parse `--urls` parameter
   - Validate URLs are accessible
4. **Idea Mode**:
   - Use `--topic` or first arg as search query
   - Note: actual research happens in Phase 3

Present mode detection results to user; confirm before proceeding.

---

### Phase 2: Topic Selection

```
Input: input mode results, user intent
Output: ARTICLE_TOPIC.md
```

**Lead-only phase** — do NOT spawn teammates.

1. Generate `ARTICLE_TOPIC.md`:
   - **Project/Topic Positioning** — what it is, what problem it solves
   - **Target Audience** — who should read this
   - **Writing Angle** — tutorial / deep dive / comparison / best practices
   - **Target Word Count** — based on `--word-count` hint (3K-10K)
   - **Key Sections (3-5)** — preliminary outline
   - **What Not to Cover** — scope boundaries

2. Present to user; confirm or edit before proceeding.

---

### Phase 3: Research + Outline

```
Input: ARTICLE_TOPIC.md, input mode
Output: research reports + ARTICLE_OUTLINE.md + STYLE_GUIDE.md
```

1. Create team `article-pipeline` with `TeamCreate`

2. **Research** (if URL Mode or Idea Mode):
   - Create task: "Gather web research using [mode] mode. Write `.work/research-*.md`."
   - Spawn 1 **article-researcher** teammate
   - Follow Shutdown Pattern

3. **Outline**:
   - Create task: "Generate ARTICLE_OUTLINE.md (3-5 sections) and STYLE_GUIDE.md (terminology, tone, format)."
   - Task must include: ARTICLE_TOPIC.md, research reports (if any), codebase path (if Repo Mode)
   - Spawn 1 **article-planner** teammate
   - Follow Shutdown Pattern

4. Present outline summary to user; confirm before writing.

---

### Phase 4: Write Sections

```
Input: ARTICLE_OUTLINE.md + STYLE_GUIDE.md + research reports + source code (if Repo Mode)
Output: `.work/sections/section-N.md`
```

1. Create `.work/sections/` directory

2. **Batch: All Sections** (not staged like book):
   - Create tasks — one per section (3-5 tasks)
   - Each task includes:
     - Section title, description, key points from ARTICLE_OUTLINE.md
     - Style guidance from STYLE_GUIDE.md
     - Research reports (for URL/Idea modes) or source paths (for Repo Mode)
   - Spawn 2-3 **article-writer** teammates IN PARALLEL
   - Prompt: "Pick up available writing tasks. Write sections per STYLE_GUIDE.md. When no tasks remain, shut down."
   - Follow Shutdown Pattern

3. **Checkpoint**: Lead reviews sections for structure compliance (quick visual check)
   - Present findings to user; confirm before proceeding

---

### Phase 5: Review + Proofread (Parallel)

```
Input: all section files `.work/sections/*.md`
Output: `.work/review-article.md` + `.work/proofread-article.md`
```

**2 teammates in parallel** — not sequential.

1. **Review**:
   - Create task: "Review article structure, fact accuracy, link validity. Write `.work/review-article.md`."
   - Spawn 1 **article-reviewer** teammate

2. **Proofread**:
   - Create task: "Proofread article for typos, punctuation, formatting, terminology. Write `.work/proofread-article.md`."
   - Spawn 1 **article-proofreader** teammate

3. Follow Shutdown Pattern (wait for both to complete)

4. **Verdict handling**:
   - **PASS**: Proceed to Phase 6
   - **FAIL**: List issues, spawn 1 article-writer for rework (max 1 round), re-run Phase 5

---

### Phase 6: Synthesis

```
Input: sections + review + proofread reports
Output: article-final.md
```

1. Create task: "Fix P0/P1 issues from review reports, compile article-final.md."
2. Spawn 1 **article-editor** teammate
3. Follow Shutdown Pattern
4. Present synthesis results (fix count, word count)

---

### Phase 7: Delivery

```
Input: article-final.md
Output: statistics + file paths
```

1. Show statistics: `wc -l`, `wc -c`, section count
2. Deliver files:
   - `article-final.md`
   - Research reports (`.work/research-*.md`)
3. Display pipeline summary

---

## Failure Handling

- **Topic Selection** → topic unsuitable; explain to user
- **Research** → insufficient sources; report gaps to user
- **Review FAIL** → rework, max 1 round
- **Synthesis** → record all fixes; display in final report

---

## Progress Reporting

After each phase:
```
✅ Phase N: [Name] — Complete
  - [Key Output]
  - [Duration / Metrics]
```
