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

Claude Code has a goal command. The user types `/goal` followed by a condition, and the session keeps working until the condition holds.

- After each turn, a separate small model judges whether the condition is met. It calls no tools and sees only the conversation, so it can judge only evidence that was stated there. Put each check and its result in your reply: a claim with no evidence in the conversation cannot satisfy the condition.
- The goal clears when the condition is met, when the judge finds it impossible to satisfy, when a turn fails on an error the user has to fix, or when the user runs `/goal clear`. A new `/goal` replaces the active one.
- A goal that was still active when a session ended is restored when the session is resumed.

`/goal` is the user's to set. Propose the condition text, written so that it can be judged from the conversation: "the test suite passes and its output is shown", not "the feature works".

The goal command holds one condition, not the plan. Keep the rest durable in:

- the repository's own tracker where it has one, such as a `backlog/` folder of tickets;
- a committed file, for anything that must outlive the session;
- `TaskCreate` / `TaskUpdate` / `TaskList`, where the session has those tools, for the execution checklist.

`EnterPlanMode` / `ExitPlanMode` serve the understanding gate and approval. `ExitPlanMode` approval **is** the explicit user approval this reference requires before starting. Use `AskUserQuestion` to resolve requirements *before* finalizing the plan; never use it to ask whether the plan is ready, since the user cannot see the plan until `ExitPlanMode`.

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

1. Record the approved objective in the most durable tracker available, as listed under the mechanism above.
2. Record acceptance criteria as the working completion standard.
3. Identify constraints, non-goals, risks, and verification requirements.
4. Build a concise execution checklist.
5. Read source artifacts before making project-specific claims.

Do not invent project facts from memory.

## Ownership

The user owns the objective, the acceptance criteria, and the decision that the goal is complete. A change to the objective or to a criterion goes to the user.

The main thread owns product understanding, the plan, status, and the evidence that each criterion is met.

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

Report a goal as met only when every approved acceptance criterion has a named check and its result, and required verification has passed or been explicitly waived by the user. A criterion you could not exercise is not met: say so.

Reporting a goal as met does not close it. The user closes it: directly, or through the `/goal` condition they set, which the judge applies to the evidence in the conversation. Mark a ticket or a tracker item done only when the user asks for that item.

Do not report a goal as met merely because:

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
