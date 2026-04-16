# Mode Consistency Verification Report

## Planner: skills/planner/SKILL.md vs agents/planner.md

### Mode Comparison Table

| Skill Mode | Agent Mode | Output Files | Consistent? |
|------------|------------|--------------|-------------|
| `book-outline` | `full-plan` | BOOK_PLAN.md, STYLE_GUIDE.md, EDITORIAL_PLAN.md | **NO** |
| `article-outline` | `article-outline` | ARTICLE_OUTLINE.md, STYLE_GUIDE.md | YES |
| `dependencies` | `dependencies` | DEPENDENCIES.md | YES |
| `code-index` | `code-index` | CODE_INDEX.md | YES |

### Inconsistency Found

**Issue**: The skill defines `book-outline` but the agent defines `full-plan` for the same functionality (Phase 3 book outline generation).

**Impact**: Pipeline orchestrator must know which name to use when invoking. If pipeline uses skill's `book-outline`, the agent's `full-plan` mode won't be triggered correctly.

**Recommended Fix**: Unify to **one name**. Options:
1. Use `book-outline` everywhere (more specific, matches `article-outline` naming pattern)
2. Use `full-plan` everywhere (emphasizes it generates 3 files vs outline-only)

**Suggestion**: Go with `book-outline` for consistency with `article-outline`.

---

## Proofreader: skills/proofreader/SKILL.md vs agents/proofreader.md

### Mode Comparison Table

| Skill Mode | Agent Mode | Output Files | Consistent? |
|------------|------------|--------------|-------------|
| `first-proofread` | `book-first` | .work/proofread-1.md | **NO** |
| `second-proofread` | `book-second` | .work/proofread-2.md | **NO** |
| `readability-pass` | `book-readability` | .work/proofread-3.md | **NO** |
| `article-proofread` | `article` | .work/proofread-article.md | **NO** |

### Inconsistencies Found

**Issue**: All 4 mode names differ between skill and agent, despite identical output file paths and functionality.

**Pattern Analysis**:
- Skill uses verbose naming: `*-proofread` suffix, `readability-pass`
- Agent uses concise naming: `book-*` prefix, `article` alone

**Impact**: Same as planner — pipeline must know which naming convention to use.

**Recommended Fix**: Unify to **one naming convention**. Options:
1. Use skill's naming (`first-proofread`, `second-proofread`, `readability-pass`, `article-proofread`)
2. Use agent's naming (`book-first`, `book-second`, `book-readability`, `article`)

**Suggestion**: Go with agent's naming (`book-*` / `article`) because:
- Shorter, easier to type in task assignments
- Clearer pipeline context (book vs article mode)
- Matches the "Mode Selection" table style in other agents

---

## Summary

| Component | Total Modes | Consistent | Inconsistent |
|-----------|-------------|------------|--------------|
| Planner | 4 | 3 | 1 (`book-outline` vs `full-plan`) |
| Proofreader | 4 | 0 | 4 (all modes differ) |

**Total: 5 of 8 modes are inconsistent (62.5% inconsistency rate)**

---

## Action Items

1. **Planner**: Decide on `book-outline` vs `full-plan`, update both skill and agent
2. **Proofreader**: Decide on naming convention, update both skill and agent
3. **Pipeline**: Verify pipeline orchestrator (skills/book-pipeline/SKILL.md) uses the unified mode names
