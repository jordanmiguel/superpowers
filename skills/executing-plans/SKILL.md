---
name: executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---

## CRITICAL CONSTRAINTS

**You MUST NOT call `EnterPlanMode` or `ExitPlanMode` during this skill.** This skill operates in normal mode, executing a plan that already exists on disk. Plan mode is unnecessary and dangerous here — it restricts Write/Edit tools needed for implementation.

# Executing Plans

## Overview

Load plan, review critically, execute tasks in batches, report for review between batches.

**Core principle:** Batch execution with checkpoints for architect review.

**Announce at start:** "I'm using the executing-plans skill to implement this plan."

## The Process

### Step 0: Load Persisted Tasks

1. Call `TaskList` to check for existing native tasks
2. **CRITICAL - Locate tasks file:** Try `<plan-path>.tasks.json`, if not found glob for matching `.tasks.json`
3. If tasks file exists AND native tasks empty: recreate from JSON using TaskCreate, restore blockedBy with TaskUpdate
4. If native tasks exist: verify they match plan, resume from first `pending`/`in_progress`
5. If neither: proceed to Step 1b to bootstrap from plan

Update `.tasks.json` after every task status change.

### Step 1: Load and Review Plan
1. Read plan file fully
2. Review critically - identify any questions or concerns about the plan
3. If concerns: Raise them with your human partner before starting
4. If no concerns: Proceed to task setup

### Step 1b: Bootstrap Tasks from Plan (if needed)

If TaskList returned no tasks or tasks don't match plan:

1. Parse the plan document for `## Task N:` or `### Task N:` headers
2. For each task found, use TaskCreate with:
   - subject: The task title from the plan
   - description: Full task content including steps, files, acceptance criteria
   - activeForm: Present tense action (e.g., "Implementing X")
3. **CRITICAL - Dependencies:** For EACH task that has blockedBy in the plan or .tasks.json:
   - Call `TaskUpdate` with `taskId` and `addBlockedBy: [list-of-blocking-task-ids]`
   - Do NOT skip this step - dependencies are essential for correct execution order
4. Call `TaskList` and verify blockedBy relationships show correctly (e.g., "blocked by #1, #2")

### Step 2: Execute Batch
**Default: First 3 tasks**

For each task:
1. Mark as in_progress
2. Follow each step exactly (plan has bite-sized steps)
3. Run verifications as specified
4. Mark as completed

### Step 3: Report
When batch complete:
- Show what was implemented
- Show verification output
- Say: "Ready for feedback."

### Step 4: Continue
Based on feedback:
- Apply changes if needed
- Execute next batch
- Repeat until complete

### Step 5: Complete Development

After all tasks complete and verified:
- Announce: "I'm using the finishing-a-development-branch skill to complete this work."
- **REQUIRED SUB-SKILL:** Use superpowers-jordan:finishing-a-development-branch
- Follow that skill to verify tests, present options, execute choice

### Team Mode

Before executing the first batch, check if the `TeamCreate` tool is available. If it is, ask the user: "Agent teams are available. Would you like to parallelize tasks within batches, or proceed with standard sequential execution?" If not available or the user declines, use the standard sequential flow.

**Note:** `TeamCreate`, `TaskCreate`, `TaskUpdate`, `TaskList`, `SendMessage`, and `TeamDelete` are Claude Code built-in tools provided by the runtime as part of the teams API (currently in beta).

**What changes:**
- Step 2 (Execute Batch): Instead of executing 3 tasks sequentially, assign batch tasks to team members working in parallel
- Step 3 (Report): Wait for all batch members to complete, then report combined results
- Steps 1, 4, 5: Unchanged (plan review, feedback loop, and completion stay the same)

**Independence constraint:** Only parallelize tasks within a batch if they are independent (touch different files, no implicit dependencies). If tasks in a batch may conflict, fall back to sequential execution for that batch.

**What doesn't change:**
- Batch boundaries and human review checkpoints remain
- Default batch size is still 3 tasks
- "Ready for feedback" checkpoint after each batch
- The human-in-the-loop approval between batches is preserved

**Team lifecycle:**
1. `TeamCreate` once before the first batch
2. For each batch: assign tasks to team members, wait for all to complete, report results, wait for feedback
3. Reuse the same team across batches (no need to recreate)
4. `TeamDelete` after the final batch completes and all work is done

**Cross-platform note:** Team mode requires Claude Code with teams enabled (beta). On Codex, OpenCode, or Claude Code without teams, use the standard sequential batch execution.

## When to Stop and Ask for Help

**STOP executing immediately when:**
- Hit a blocker mid-batch (missing dependency, test fails, instruction unclear)
- Plan has critical gaps preventing starting
- You don't understand an instruction
- Verification fails repeatedly

**Ask for clarification rather than guessing.**

## When to Revisit Earlier Steps

**Return to Review (Step 1) when:**
- Partner updates the plan based on your feedback
- Fundamental approach needs rethinking

**Don't force through blockers** - stop and ask.

## Remember
- Review plan critically first
- Follow plan steps exactly
- Don't skip verifications
- Reference skills when plan says to
- Between batches: just report and wait
- Stop when blocked, don't guess
- Never start implementation on main/master branch without explicit user consent

## Integration

**Required workflow skills:**
- **superpowers-jordan:writing-plans** - Creates the plan this skill executes
- **superpowers-jordan:finishing-a-development-branch** - Complete development after all tasks
