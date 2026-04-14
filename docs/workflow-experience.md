# Source Code Deep Analysis Book — Workflow Experience Summary

> Based on real-world experience from the Hermes Agent publishing pipeline

## Architect Review Notes

- **Agent Teams architecture is sound** — 5 Writers divided by Part avoids content overlap
- **Three reviews and proofreading passes are separated** — Structure checks, consistency, and readability are three independent dimensions
- **Style guide upfront** — Unify terminology before writing to prevent late-stage rework
- **Suggested improvement** — Add a "chapter dependency graph" before writing to establish cross-reference relationships
- **Suggested improvement** — The consolidation agent should collect all reports first before acting, rather than reading and editing incrementally

## Editor-in-Chief Review Notes

- **Chapter structure template is effective** — Opening metaphor + Mermaid + deep dive + decision box + reflection + principles
- **Unified design decision box format** — Decision / alternatives / trade-offs / rationale is a solid pattern
- **Content overlap handling** — The home-chapter-plus-cross-reference strategy is correct but must be agreed upon before writing
- **Suggested improvement** — Add "difficulty level" markers to help readers gauge chapter difficulty
- **Suggested improvement** — Answers to "Stop and Think" questions should be kept for the Editor-in-Chief's reference, not deleted
- **Suggested improvement** — Appendices must be placed after the main text during consolidation — an easy detail to get wrong

---

## 1. Overall Architecture

### Correct Architecture

```
Main Agent (Coordinator)
├── book-planner → Outline / style guide / editorial pipeline
├── 5 Writers (parallel) → First drafts
├── 3 Reviewers (parallel) → Initial + re-review + verification
├── 3 Proofreaders (parallel) → First pass + second pass + third pass
├── Cover Designer → Cover design
├── Preface Writer → Preface writing
├── Editor-in-Chief → Consolidation
└── Appendices Writer → Appendix composition
```

### Key Disciplines

1. **Main Agent never writes chapters manually** — Only coordinates, summarizes, and delegates
2. **Everything that can run in parallel must** — First pass, second pass, third pass, cover, and preface have no dependencies; launch simultaneously
3. **Long tasks must report progress mid-flight** — The main agent should not wait idle for all subagents to finish before reporting
4. **Review immediately after writing, don't batch** — Start a reviewer as soon as each writer completes

## 2. Three Reviews and Three Proofreading Passes

### Three Reviews (Structure + Consistency)

| Review Level | Checks | Automation Level |
|--------------|--------|-----------------|
| Initial review | Chapter structure, format compliance, source code accuracy | Semi-automatic (grep + manual verification) |
| Re-review | Cross-chapter consistency, terminology unification, content deduplication | Manual (requires semantic understanding) |
| Final review | Overall book quality, completeness, readability | Manual |

### Three Proofreading Passes (Text + Technical + Readability)

| Pass | Checks | Automation Level |
|------|--------|-----------------|
| First pass | Typos, punctuation, formatting, terminology | Semi-automatic (grep-assisted) |
| Second pass | Cross-references, content overlap, design decision contradictions | Manual (requires cross-chapter comparison) |
| Third pass | Read-through, narrative coherence, pacing, tone consistency | Manual (pure reading experience) |

### Lessons Learned

- **All proofreading passes should launch in parallel** — We made the mistake of waiting serially
- **Initial review can be automated** — ASCII detection, Mermaid counting, structure checks can all use grep
- **Re-review must wait until all first drafts are complete** — Requires a global view
- **Proofreading reports should include priorities** — P0/P1/P2 gives consolidation a clear fix order

## 3. Style Consistency

### Must Create Before Writing

1. **STYLE_GUIDE.md** — Glossary, chapter template, prohibitions, content overlap mapping
2. **BOOK_PLAN.md** — Chapter outline, per-chapter content description
3. **EDITORIAL_PLAN.md** — Role assignments, pipeline stages, progress tracking

### Most Common Style Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| ASCII art vs Mermaid | Writer did not follow guidelines | Verifier auto-detects; fails if not compliant |
| Inconsistent terminology | Different Writers use different words | STYLE_GUIDE.md glossary; unify during consolidation |
| Decision box format varies | Some use blockquote, some use HTML | Specify format in STYLE_GUIDE.md; unify during consolidation |
| Tutorial vs architecture analysis | Writer leans toward tutorial style | Prohibitions in STYLE_GUIDE.md; reviewer checks |
| Content duplication | No clear home chapter for concepts | Content overlap mapping table; agree before writing |

## 4. Agent Teams Best Practices

### Main Agent Responsibilities

- Coordinate progress
- Summarize reports
- Delegate tasks
- Report to user
- **Does not write chapters, review, or proofread**

### Writer Independence

- Each Writer is responsible only for their assigned chapters
- Receives STYLE_GUIDE.md and BOOK_PLAN.md
- Content overlap handled per mapping table
- Goes idle automatically after completion, awaiting reviewer feedback

### Reviewer Automation

- Structure checks can be automated with grep
- Code references require actual verification
- Terminology consistency can be semi-automatically checked
- Report format should be standardized

### Editor-in-Chief Fix Order

- Read all reports first (review + proofread + consistency)
- Fix in P0/P1/P2 priority order
- Complete P0 fixes before moving to P1
- Compile final manuscript last

## 5. Pipeline Stage Dependencies

```
Planning (sequential)
  ↓
First Draft (5 writers in parallel)
  ↓
Three Reviews (initial + re-review in parallel; final review after all initial reviews complete)
  ↓
Rework (writers revise based on review feedback)
  ↓
Verification (verifier automated checks)
  ↓
Three Proofreading Passes (first + second + third + cover + preface — all parallel)
  ↓
Consolidation (Editor-in-Chief)
  ↓
Appendices (Appendices Writer)
  ↓
Final Manuscript Compilation (appendices must come after main text)
```

## 6. Improvements After Plugin-ization

Compared to a manual pipeline, the plugin provides:

1. **Automatic stage progression** — No need to manually wait for results before dispatching the next agent
2. **Automatic progress feedback** — User notified automatically at each stage completion
3. **Automatic retry on failure** — Writer rework triggers automatic re-verification
4. **Built-in experience** — STYLE_GUIDE.md template already contains best practices
5. **Reusable for any project** — Not limited to Hermes; works with any open-source codebase
