# Broken File References Report

**Date**: 2026-04-16
**Purpose**: Check for references to deleted skill files after the skill merge

## Deleted Files Being Checked

| File | Status | Notes |
|------|--------|-------|
| `skills/book-planner/SKILL.md` | DELETED | Merged into `planner` |
| `skills/article-planner/SKILL.md` | DELETED | Merged into `planner` |
| `skills/book-proofreader/SKILL.md` | DELETED | Merged into `proofreader` |
| `skills/article-proofreader/SKILL.md` | DELETED | Merged into `proofreader` |

---

## Broken References Found

### 1. `.claude-plugin/plugin.json` (Line 28)

```json
"book-proofreader",
```

**Issue**: References deleted `book-proofreader` skill
**Fix**: Replace with `"proofreader"`

---

### 2. `CHANGELOG.md` (Lines 12-13)

```markdown
- 5 skills: book-planner, book-writer-template, book-consistency-reviewer, book-proofreader, book-pipeline (pipeline orchestration)
- 11 agents: book-planner (dynamic chapters), 5 writers (foundation/core-loop/core-system/tools/integration), book-chapter-reviewer (structure review), book-technical-reviewer (fact-checking), book-verifier, book-editor-in-chief, book-preface-writer
```

**Issue**: References deleted `book-planner` and `book-proofreader` skills
**Fix**: Update to reflect merged skill names (`planner`, `proofreader`)

---

### 3. `SECURITY.md` (Line 24)

```markdown
- The `book-planner` skill may reference the `understand` skill if available (optional external plugin)
```

**Issue**: References deleted `book-planner` skill
**Fix**: Replace with `planner`

---

### 4. `CLAUDE.md` (Lines 33, 36, 115, 143)

```markdown
Line 33: │   ├── book-planner/SKILL.md    # Planner: analyze codebase, draft outline, style guide
Line 36: │   └── book-proofreader/SKILL.md # Three-pass proofreading (first/second/readability modes)
Line 115: **Problem**: `book-proofreader` had all three proofreading passes in one SKILL.md with no mode selection
Line 143: **Validation**: `mmdc -i file.mmd -o /dev/null` is mandatory in Phase 8 (book-verifier), Phase 9 (book-proofreader first pass), and Phase 10.5 (book-final-reviewer).
```

**Issue**: Lines 33, 36 reference deleted files in directory structure. Lines 115, 143 are historical documentation (acceptable to keep as-is for context).
**Fix**: Update directory structure on lines 33, 36 to reflect merged skills:
- `book-planner/SKILL.md` → `planner/SKILL.md`
- `book-proofreader/SKILL.md` → `proofreader/SKILL.md`

Lines 115 and 143 can remain as historical context (they describe past problems that were fixed).

---

### 5. `README.md` (Lines 61, 64)

```markdown
Line 61: │   ├── book-planner/            # Planner: analyze codebase, draft outline, style guide
Line 64: │   ├── book-proofreader/        # Proofreading: text / cross-references / readability
```

**Issue**: Directory structure references deleted skills
**Fix**: Update to:
- `book-planner/` → `planner/`
- `book-proofreader/` → `proofreader/`

---

### 6. `agents/proofreader.md` (Lines 28, 76)

```markdown
Line 28: Refer to `skills/book-proofreader/SKILL.md` for detailed checklists.
Line 76: Refer to `skills/article-proofreader/SKILL.md` for detailed checklist.
```

**Issue**: References deleted skill files
**Fix**: Update to:
- Line 28: `skills/proofreader/SKILL.md`
- Line 76: `skills/proofreader/SKILL.md` (with note about article mode)

---

### 7. `docs/workflow-experience.md` (Line 30)

```markdown
├── book-planner → Outline / style guide / editorial pipeline
```

**Issue**: References deleted `book-planner` skill
**Fix**: Replace with `planner`

---

### 8. `.work/article-pipeline-review.md` (Lines 16, 18, 31, 56)

```markdown
Line 16: | `skills/article-planner/SKILL.md` | ✅ EXISTS |
Line 18: | `skills/article-proofreader/SKILL.md` | ✅ EXISTS |
Line 31: | `agents/article-proofreader.md` | ✅ EXISTS | ✅ Valid | ✅ Read, Write, Grep, Glob, Bash |
Line 56: | `skills` array | article-pipeline, article-researcher, article-planner, article-reviewer, article-proofreader | ✅ ALL PRESENT (lines 28-32) |
```

**Issue**: This is a review report file that incorrectly marks deleted files as "EXISTS"
**Fix**: This file appears to be a stale/outdated review report. Either:
1. Delete and regenerate the review report
2. Update to reflect current state: `planner` and `proofreader` are the correct skill names

Note: `agents/article-proofreader.md` does NOT exist - the correct agent is `proofreader.md` with article mode.

---

## Files Already Correct

### `skills/book-pipeline/SKILL.md`

References to `planner` and `proofreader` are already correct:
- Line 247: `planner` with `code-index` mode
- Line 131: `planner` with `article-outline` mode
- Line 177: `proofreader` with `article` mode
- Lines 415-418: `proofreader` with book modes

No broken references found.

### `skills/article-pipeline/SKILL.md`

All references use `planner` and `proofreader` correctly. No broken references found.

### `skills/article-pipeline-reviewer/SKILL.md`

All references use `planner` and `proofreader` correctly. No broken references found.

---

## Summary of Required Fixes

| File | Lines | Broken Reference | Suggested Fix |
|------|-------|------------------|---------------|
| `.claude-plugin/plugin.json` | 28 | `book-proofreader` | Replace with `proofreader` |
| `CHANGELOG.md` | 12-13 | `book-planner`, `book-proofreader` | Update to `planner`, `proofreader` |
| `SECURITY.md` | 24 | `book-planner` | Replace with `planner` |
| `CLAUDE.md` | 33, 36 | `book-planner/SKILL.md`, `book-proofreader/SKILL.md` | Replace with `planner/SKILL.md`, `proofreader/SKILL.md` |
| `README.md` | 61, 64 | `book-planner/`, `book-proofreader/` | Replace with `planner/`, `proofreader/` |
| `agents/proofreader.md` | 28, 76 | `skills/book-proofreader/SKILL.md`, `skills/article-proofreader/SKILL.md` | Both → `skills/proofreader/SKILL.md` |
| `docs/workflow-experience.md` | 30 | `book-planner` | Replace with `planner` |
| `.work/article-pipeline-review.md` | 16, 18, 31, 56 | Multiple | Regenerate or update with current skill names |

---

## Verification Commands

After making fixes, verify with:

```bash
# Check for any remaining references to deleted skills
grep -rn "book-planner\|article-planner\|book-proofreader\|article-proofreader" --include="*.md" --include="*.json" .

# Verify plugin.json skills array matches existing files
cat .claude-plugin/plugin.json | jq '.skills[]' | while read skill; do
  if [ -f "skills/$skill/SKILL.md" ]; then
    echo "✓ $skill"
  else
    echo "✗ $skill (MISSING)"
  fi
done
```
