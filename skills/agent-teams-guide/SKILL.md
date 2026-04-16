---
name: agent-teams-guide
description: Reference guide for Agent Teams usage in tech-insider pipeline. Use when spawning teammates, managing shutdown, or debugging team lifecycle issues. Covers lead execute loop, shutdown protocol, task distribution, prohibited operations.
---

# Agent Teams Guide — tech-insider

This skill documents **correct usage patterns** for Agent Teams in tech-insider pipeline. Read when:
- Spawning teammates for new phase
- Managing shutdown between phases
- Debugging "active members" errors on `TeamDelete`
- Uncertain about lead vs teammate responsibilities

---

## Core Principles

### 1. Lead's Execute Loop

```
Decide → SendMessage → YIELD → Wait for mailbox → Repeat
```

**Critical**: `SendMessage` to teammate is **always last action** of turn. Any tool call after blocks mailbox, causes **deadlock**.

**Correct sequence**:
1. Decide: "Spawn 4 reviewers for Phase 6a"
2. Create tasks via `TaskCreate`
3. Spawn 4 teammates via `Agent` (parallel)
4. **SendMessage** to assign tasks
5. **YIELD** — no more tool calls
6. Wait for `idle_notification` or task completion in mailbox
7. Repeat

**Incorrect** (causes deadlock):
```
SendMessage to teammate → Bash("git status") → Read output
```

---

## Team Lifecycle

### Golden Rules

| Rule | Description |
|------|-------------|
| **ONE team** | Create team once at Phase 3, recycle members between phases |
| **No subagent fallback** | Never use `Agent` without `team_name` when teammates encounter errors |
| **TeamDelete at end only** | Call `TeamDelete` only at Phase 11 (Delivery) |

### Phase Transition Pattern

Between every phase that uses Agent Teams:

1. **Shutdown old teammates**:
   ```
   SendMessage({
     to: "<teammate-name>",
     summary: "shutdown",
     message: {
       type: "shutdown_request",
       request_id: "shutdown-1",
       approve: true
     }
   })
   ```
   - One request per teammate — never race multiple requests to same teammate
   - Use structured message format above — NOT plain string

2. **YIELD IMMEDIATELY** after each `SendMessage`
   - Turn over
   - Do NOT proceed to step 3 in same turn

3. **Wait for all shutdowns**
   - Teammates confirm via `shutdown_response` in mailbox
   - System takes moment to process
   - Do NOT proceed until all old teammates confirmed gone (`idle_notification` received)

4. **Create tasks** for next phase via `TaskCreate`

5. **Spawn ALL teammates at once**
   - Call `Agent` multiple times in parallel (one per teammate)
   - ALL with same `team_name: "book-pipeline"` (or current team name)
   - Do NOT spawn one at a time sequentially

6. **YIELD IMMEDIATELY** after spawning

7. **Wait** — teammates auto-claim tasks, complete them, auto-claim next

8. **Shutdown when idle** — when all tasks done, send ONE structured shutdown request per teammate

---

## Lead's Role

**Lead does NOT execute work.** All execution, review, validation delegated to teammates via `SendMessage`.

| Lead Does | Lead Does NOT |
|-----------|---------------|
| Break down tasks | Clone repositories |
| Define scope | Read source code |
| Create task definitions | Write chapter content |
| Spawn teammates | Review chapters |
| Manage shutdown lifecycle | Run `mmdc` validation |
| Present results to user | Run `Grep` for task-related work |

**Exception**: User-facing commands like `git commit`, `git push` are lead responsibilities.

---

## Task Distribution

| Pattern | When to Use | How |
|---------|-------------|-----|
| **Auto-claim** | Homogeneous tasks (4 reviewers reviewing 16 chapters) | Create tasks via `TaskCreate` BEFORE spawning. Teammates auto-claim from shared list. After finishing, auto-claim next. |
| **Explicit spawn** | Heterogeneous tasks (3 proofread passes with different modes) | Each teammate gets different prompt/mode. Spawn explicitly with mode instruction. |

---

## Shutdown Protocol (4-Step Sequence)

**Step 1**: Send shutdown_request to each teammate (one at a time, yield after each)
```
SendMessage({
  to: "reviewer-1",
  summary: "shutdown",
  message: { type: "shutdown_request", request_id: "shutdown-1", approve: true }
})
→ YIELD
```

**Step 2**: Wait for `shutdown_response` confirmation from each teammate
```
Mailbox: { type: "shutdown_response", request_id: "shutdown-1", approve: true }
```

**Step 3**: Confirm ALL teammates have responded before proceeding
- Check `idle_notification` from all teammates
- Do NOT assume — wait for explicit confirmation

**Step 4**: Only after ALL confirmations → proceed to next phase or `TeamDelete`

---

## State Sync Gotcha

**Problem**: Framework delays shutdown confirmation in team directory. `TeamDelete` fails with "active member" if called too soon.

**Solution**: Always wait for **explicit** `shutdown_response` from every teammate before calling `TeamDelete`.

**Pattern**:
```
SendMessage(shutdown_request)
  ↓ YIELD
Wait for idle_notification + shutdown_response
  ↓ YIELD
Spawn next teammates (if continuing)
  ↓ YIELD
Wait for completion
  ↓ YIELD
TeamDelete (only at very end)
```

---

## Prohibited Operations

| Prohibited | Why | Correct Approach |
|------------|-----|------------------|
| Manually edit/delete `~/.claude/teams/` or `~/.claude/tasks/` | Bypasses protocol, causes state desync | Use `TeamDelete` only |
| Use `sleep` loops or `Monitor` to poll teammates | Blocks mailbox — deadlock risk | Wait for mailbox notifications |
| Race multiple `shutdown_request` to same teammate | Confuses shutdown state | One request per teammate, yield after each |
| Continue working after `SendMessage` | Blocks mailbox — deadlock | `SendMessage` is ALWAYS last action of turn |
| Fall back to `Agent` without `team_name` | Breaks team model | Fix issue within team or report to user |

---

## Examples

### ✅ Correct: Spawning Reviewers (Phase 6a)

```
1. TaskCreate (4 tasks, one per chapter)
2. Agent × 4 (parallel, all with team_name: "book-pipeline")
3. SendMessage to each reviewer with task assignment
4. YIELD — wait for mailbox
5. Receive idle_notification from each reviewer when done
6. SendMessage(shutdown_request) to each reviewer
7. YIELD after each shutdown_request
8. Wait for shutdown_response from all
9. Proceed to Phase 6b
```

### ❌ Incorrect: Polling After SendMessage

```
SendMessage to reviewer → Bash("sleep 5 && cat output") → Read result
```

**Why wrong**: `Bash` after `SendMessage` blocks mailbox. Reviewer's completion message cannot be delivered — deadlock.

### ✅ Correct: Phase Transition

```
1. SendMessage(shutdown_request) to reviewer-1 → YIELD
2. Wait for shutdown_response from reviewer-1
3. SendMessage(shutdown_request) to reviewer-2 → YIELD
4. Wait for shutdown_response from reviewer-2
5. TaskCreate for next phase
6. Agent × N (parallel spawn)
7. YIELD
```

### ❌ Incorrect: Manual Cleanup

```
Bash("rm -rf ~/.claude/teams/doc-verify")
```

**Why wrong**: Bypasses shutdown protocol. Team state desyncs. Future `TeamDelete` calls may fail.

---

## Memory Context

This skill created from feedback captured on 2026-04-16 after violating shutdown protocol during `doc-verify` team:
- Sent shutdown_request → then called `TeamDelete` → failed with "active members"
- Sent shutdown_request → then ran `Bash` to check status → blocked mailbox
- Manually deleted `~/.claude/teams/doc-verify` with `rm -rf` → bypassed protocol

**Memory file**: `~/.claude/projects/-Users-yywl-coding-tech-insider/memory/feedback_agent_teams.md`

---

## Quick Reference Card

| Do | Don't |
|----|-------|
| `SendMessage` → YIELD | `SendMessage` → more tool calls |
| Wait for `shutdown_response` | Assume shutdown is done |
| One shutdown_request per teammate | Race multiple requests to same teammate |
| `TeamDelete` only at Phase 11 | `TeamDelete` between phases |
| Delegate work via `SendMessage` | Lead executes work directly |
| Use `team_name` on all `Agent` calls | Fall back to subagents without team |

---

## Related Files

- `skills/book-pipeline/SKILL.md` — Full pipeline orchestration with Agent Teams
- `skills/article-pipeline/SKILL.md` — Article pipeline (simplified flow)
- `~/.claude/CLAUDE.md` section 5 — Original Agent Teams guidelines
