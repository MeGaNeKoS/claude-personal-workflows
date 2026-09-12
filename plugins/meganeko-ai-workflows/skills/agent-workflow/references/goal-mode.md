# Goal Mode

Goal mode is an explicit operating contract with the user. Use it for outcome ownership, not for small one-shot tasks.

## When To Propose Goal Mode

Propose goal mode when the task is likely to require sustained execution across multiple turns or work areas, such as:

- product or system implementation with unclear remaining scope
- cross-cutting frontend/backend/infrastructure work
- repeated implementation, verification, and adjustment loops
- user acceptance criteria or product scope tracking
- long-running debugging or migration work
- work where status continuity matters

Do not start goal mode just because the task is complex. Propose it only when durable objective tracking would materially improve the work.

## Mechanism In Claude Code

Claude Code has no single goal primitive. Assemble the durable tracker from:

- `EnterPlanMode` / `ExitPlanMode` for the understanding gate and approval. `ExitPlanMode` approval **is** the explicit user approval this reference requires before starting. Use `AskUserQuestion` to resolve requirements *before* finalizing the plan; never use it to ask whether the plan is ready, since the user cannot see the plan until `ExitPlanMode`.
- `TaskCreate` / `TaskUpdate` / `TaskList` for the execution checklist and for status that survives compaction.
- Committed files and git history for verification evidence.

Anything held only in conversation history is not durable. Record it.

## Approval Rule

The agent may propose goal mode, but must not start it unless the user explicitly approves the inferred goal and acceptance criteria.

A proposal must include:

- inferred objective
- why goal mode fits
- proposed acceptance criteria
- known assumptions or unknowns
- what the agent would track

If the user does not approve, continue in normal mode.

## Understanding Gate

Before starting goal mode, demonstrate enough understanding for the user to judge whether the goal is correct.

Cover:

- intended outcome
- users or audience
- system boundaries
- runtime/deployment context when relevant
- success criteria
- known unknowns

If the user corrects the understanding, revise it before starting goal mode.

## Start Checklist

When goal mode is approved:

1. Record the approved objective with `TaskCreate`, and in a committed file when the goal spans sessions.
2. Record acceptance criteria as the working completion standard.
3. Identify constraints, non-goals, risks, and verification requirements.
4. Build a concise execution checklist.
5. Read source artifacts before making project-specific claims.

Do not invent project facts from memory.

## Ownership

The main thread owns product understanding, architecture judgment, acceptance criteria, status, and final completion decisions.

Checklists, tools, and subagents may support execution, but they do not replace main-thread judgment.

## Execution Loop

For each work slice:

1. Gather only the source context needed for that slice.
2. Choose the smallest coherent implementation or research step.
3. Make scoped changes.
4. Run the relevant checks.
5. Deploy or test in the intended runtime when required.
6. Report what changed, what passed, what failed, and what remains.

Keep status concrete. Avoid vague progress language.

## Completion Rule

Mark a goal complete only when the approved acceptance criteria are met and required verification has passed or been explicitly waived by the user.

Do not mark complete merely because:

- implementation is partially done
- context is low
- the work is difficult
- tests were skipped
- the user has not yet audited
- a temporary workaround exists

If verification could not be run, report the residual risk.

## Blocked Rule

Declare blocked only when the agent cannot make meaningful progress without user input or an external state change.

Before declaring blocked:

- inspect relevant files and docs
- run safe diagnostics
- check logs/config/scripts when applicable
- isolate the exact missing condition

State blockers as concrete requirements.
