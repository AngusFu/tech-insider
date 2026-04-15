# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**tech-insider** is a Claude Code plugin that automates publication-grade deep source-code analysis books. From cloning a repo to delivering a 60K+ word technical manuscript — 5 parallel writers, 3 review rounds, 3 proofreading passes, Editor-in-Chief synthesis, fully automated.

Born from a real publishing project for [Hermes Agent](https://github.com/NousResearch/hermes-agent) (50K+ stars, MIT, 67K-word, 16-chapter book).

## Quick Start

```bash
# Local testing
claude --plugin-dir /Users/yywl/coding/tech-insider

# Via marketplace (if installed)
/plugin install tech-insider@tech-insider-marketplace

# Run the pipeline
/tech-insider:make-book https://github.com/NousResearch/hermes-agent --title "My Book Title"
```

## Directory Structure

```
tech-insider/
├── .claude-plugin/
│   ├── plugin.json              # Plugin manifest (name, skills, commands)
│   └── marketplace.json         # Marketplace registration
├── skills/
│   ├── book-pipeline/SKILL.md   # Pipeline orchestration — full flow, Agent Teams
│   ├── book-planner/SKILL.md    # Planner: analyze codebase, draft outline, style guide
│   ├── book-writer-template/SKILL.md  # Writer template: chapter structure, writing rules
│   ├── book-consistency-reviewer/SKILL.md  # Cross-chapter consistency review
│   └── book-proofreader/SKILL.md # Three-pass proofreading (first/second/readability modes)
├── agents/
│   ├── book-writer.md          # Generic Writer: any chapter type, assigned from BOOK_PLAN.md
│   ├── book-chapter-reviewer.md  # Structure review (initial review)
│   ├── book-technical-reviewer.md# Technical fact-checking against source code
│   ├── book-verifier.md          # Automated structure + Mermaid verification
│   ├── book-final-reviewer.md    # Final review of compiled book (Phase 10.5)
│   ├── book-editor-in-chief.md   # Editor-in-Chief: P0/P1/P2 fixes + final compilation
│   └── book-preface-writer.md    # Preface writing
├── commands/
│   └── make-book.md              # /tech-insider:make-book launch command
└── docs/
    └── workflow-experience.md    # Workflow experience summary from Hermes project
```

## Plugin Architecture

- **Skills** define behaviors (`skills/<name>/SKILL.md` with YAML frontmatter: `name`, `description`, `user-invocable`)
- **Agents** are specialized workers (`agents/<name>.md` with YAML frontmatter: `name`, `description`, `allowed-tools`)
- **Commands** are slash commands (`commands/<name>.md` with YAML frontmatter: `name`, `description`, `argument-hint`)
- Plugin manifest is `.claude-plugin/plugin.json` — must include `skills` and `commands` arrays

## Pipeline Flow (11 Phases)

```
Phase 1: Clone + Analyze          → metrics, language distribution
Phase 2: Topic Selection           → TOPIC.md (worth writing?)
Phase 3: Outline                   → BOOK_PLAN.md + STYLE_GUIDE.md + EDITORIAL_PLAN.md
Phase 4: Pre-Writing Coordination  → DEPENDENCIES.md (spawn planner teammate)
Phase 4.5: Code Index              → CODE_INDEX.md (spawn planner teammate)
Phase 5: First Draft (staged)      → foundation writer first, then 4 parallel
Phase 6: Three Reviews             → initial + technical + cross-chapter + final verdict
Phase 7: Rework                    → max 2 rounds
Phase 8: Verification              → automated checks + Mermaid validation
Phase 9: Proofreads + Preface      → 3 proofreads (parallel) + preface writing
Phase 9.5: Preface Review          → accuracy/tone/scope check
Phase 10: Synthesis (5 passes)     → P0 fixes → P1 fixes → P2 fixes → appendices (1 teammate) → compile
Phase 10.5: Final Review           → book-final.md quality gate (ASCII residue, structure, cross-refs)
Phase 10.6: External Review        → compilation integrity (chapter order, completeness, duplicates, layout, TOC)
Phase 11: Delivery                 → stats + files
```

### Artifact Convention

All intermediate files go in `.work/` directory:
- `review-chXX.md` — initial review reports
- `tech-review-chXX.md` — technical review reports
- `review-consistency.md` — cross-chapter consistency
- `verification-status.md` — automated check results
- `proofread-1/2/3.md` — three proofreading reports
- `preface-review.md` — preface review
- `review-consistency.md` — cross-chapter consistency (consolidated from per-Part reports)
- `final-review-verdict.md` — go/no-go decision

## Pitfalls & Lessons Learned

### Phase 1 Must Be Sequential
Phase 1 is executed by the lead (not teammates). All Bash calls MUST be sequential — do NOT run parallel Bash tool calls. `git clone` must complete before running `find`/`wc`.

### Phase 2 Is Lead-Only, No Team
Phase 2 (Topic Selection) and Phase 1/6d/11 are lead responsibilities. Do NOT create teams or spawn teammates for these phases. If `TOPIC.md` already exists, read it instead of regenerating.

### Team Lifecycle Must Use Tools
**NEVER** manually edit or delete `~/.claude/teams/` or `~/.claude/tasks/` files. Always use `TeamCreate` → `Agent` → `TaskUpdate` → `SendMessage(shutdown)` → `TeamDelete` lifecycle.

### No Subagent Fallback
**NEVER** fall back to subagents (`Agent` without `team_name`) when teammates encounter errors. Fix the issue within the team model or report to the user.

### Team Size Hard Limit — Max 6
**Rule**: Team size MUST NOT exceed 6 active teammates at any time.
**Reviewer pattern**: Phases 6a/6b/6c/7 use exactly 4 reviewer teammates maximum regardless of chapter/Part count. Each processes multiple chapters serially via the shared task list.
**Other phases**: Pipeline (lead) manages `.work/team-queue.md`. Write all pending assignments, spawn up to 6, and when a teammate completes and shuts down, onboard the next queued one. Do NOT manually split into batches.

### Chapter Output Path
All chapters go to `.work/chapters/chXX-*.md`, NOT the book root. Writers, reviewers, and pipeline all use this path.

### Multi-Skill Mode Selection
**Problem**: `book-proofreader` had all three proofreading passes in one SKILL.md with no mode selection — all three would run every time, overwriting each other's output.
**Fix**: Added explicit invocation mode table at the top of the skill. Pipeline must pass mode context when invoking.

### Editor-in-Chief Chunked Mode Selection
**Problem**: Pipeline calls editor-in-chief 3 times (P0/P1/P2), but agent had no mode selection — would do all 3 passes every time.
**Fix**: Added mode table (`p0-fix` / `p1-fix` / `p2-fixes` / `compile-final`) and "read all reports first, do not incremental edit" discipline. Phase 10 now has 5 passes (P0 → P1 → P2 → appendices parallel → compile) instead of 3.

### Consistency Review Split by Part
Phase 6c spawns up to 4 **book-consistency-reviewer** teammates (not one for all chapters, not one per Part). Each reviewer handles their assigned Part, picks up next Part if finishes early. Pipeline consolidates part reports into `.work/review-consistency.md`.

### Re-entrancy Check
### Agent `.work/` Paths Are Contracts, Not Hacks
Paths like `.work/proofread-1.md` look "hardcoded" but they are **inter-agent contracts** — all agents (proofreader, editor-in-chief, pipeline) agree on these canonical locations. Don't try to make them dynamic; fix the invocation logic instead.

### Plugin Manifest Must Declare Everything
`plugin.json` must include `skills` and `commands` arrays listing all skill/command names. Without them, skills are invisible to Claude Code even if the files exist.

### No Duplicate Instructions
`commands/make-book.md` should only parse parameters and delegate to `skills/book-pipeline/SKILL.md`. Don't duplicate pipeline logic in the command file.

### Lead Does Not Write
The pipeline orchestrator (lead) only coordinates, delegates, and reports. All writing, reviewing, and proofreading is delegated to Agent Teams teammates. If lead starts writing chapters, context window explodes.

### Staged Writer Launch
Foundation chapters go first (set tone). After style checkpoint, remaining chapters split among up to 3 generic writers in parallel with Batch 1 output as reference.

### Mermaid Is the Only Diagram Choice
No ASCII art allowed. All architecture diagrams must be Mermaid. Verified by `book-verifier` using `mmdc -i file.mmd -o /dev/null`.

### CODE_INDEX.md Cuts Token Cost 50%+
Writers and reviewers query the pre-computed index instead of reading raw source. Drill into source only for specific citations.

### Chunked Synthesis
Phase 10 processes in 5 sequential passes: P0 fixes → P1 fixes → P2 fixes → appendices (1 editor-in-chief) → final compile. Each pass shuts down old teammate, spawns new one.

### Generic Writer Agent
Only 1 `book-writer.md` agent needed. Chapter assignments come from `BOOK_PLAN.md` at runtime. Phase 5 splits into Batch 1 (1 writer, foundation chapters) + Batch 2 (up to 3 writers, remaining chapters). Phase 7 rework spawns up to 4 generic writers for FAIL chapters.

### Proofreading Must Be Parallel
All 3 proofreading passes + preface writing have no inter-dependencies — launch simultaneously, not sequentially.

### Phase 9.5 Preface Review
Spawn a book-chapter-reviewer teammate to review the preface against TOPIC.md, BOOK_PLAN.md, and STYLE_GUIDE.md before Phase 10.

### TeamDelete Between Every Phase
**Old**: Every phase called TeamDelete → TeamCreate, causing "Already leading team" errors.
**New**: Create team `book-pipeline` ONCE at Phase 3. Between phases: shutdown old teammates → spawn new teammates (same team_name). TeamDelete only at Phase 11.

### SendMessage Requires summary for String Messages
When using `SendMessage` with a plain string message, `summary` parameter is required.
