# Subagents

Subagents extend execution capacity in normal turns and in goal mode. They do not replace main-thread understanding or final judgment.

Subagents are not goal-mode-only. Use them for ordinary coding, inspection, verification, and review work when the task is bounded and delegation would improve throughput or context hygiene.

When the user has explicitly requested subagent-based workflow, delegate bounded procedural execution by default. The main thread must state a concrete reason before doing that work directly.

Classify the next unit of work before running tools or editing files:

- Main-thread work: understanding intent, taking scope, architecture, product, and risk decisions to the user, resolving ambiguous requirements, decomposition, accepting results, and the final acceptance check.
- Subagent work: procedural execution, tool-specific operation, repetitive mechanical changes, bounded source inspection, bounded verification, cleanup after a chosen direction, implementation of a clearly defined slice, and review of a bounded diff or artifact.

If the next unit is subagent work, delegate it unless a skip condition applies.

Main-thread judgment does not absorb adjacent procedural work. Once the rule, pattern, product behavior, or architecture is decided, subagents can still gather matches, inspect broad file sets, run checks, perform mechanical edits, or verify the result.

Skip conditions:

- The work is a single quick read needed for a main-thread decision, or a single-fact lookup where the file, symbol, or value is already known.
- The `Agent` tool is unavailable in the current session.
- The task requires main-thread judgment.
- Delegation would require secrets, destructive operations, production exposure, or external side effects without explicit approval.
- The user explicitly asks the main thread to do it directly.

If skipping delegation for subagent work, state: `Skipping subagent because: <specific reason>.`

Subagents are explicit delegation. Follow the active project and runtime policy as the source of truth for whether bounded work should be delegated by default or requires user approval.

## Scope And Reuse

Give each subagent a scope it owns: a set of files for one that edits, a subject for one that only reads. Name it after that scope.

- Before every dispatch, look at the subagents the session already has. When one owns the files the task touches, send it the task, whatever the task's topic. Its knowledge of those files is the reason to reuse it; a different topic is not a reason for a new one.
- Start a new subagent only when no existing one owns the scope.
- Two subagents never edit the same files. A task that spans two owners is two briefs.
- A defect goes back to the subagent that built it.
- Wanting another look at work already reviewed is not a reason for a second reviewer. A second one is right when it needs different files or different expertise.

Reuse means addressing the same agent again: call `SendMessage` with the agent's ID or name to continue it with its context intact. A new `Agent` call with a named type starts a subagent that has none of the conversation.

A subagent that works many tasks has its context summarized like any other session, and a summary keeps the gist of a brief while dropping its numbered requirements. Put a long brief, and any brief to a subagent that has already worked several tasks, in a file, and tell the subagent to read it in full and to re-read it before reporting.

## Lifecycle

Claude Code subagents are **not persistent workers occupying a pool**. There is no slot to free and nothing to close, so no cleanup step gates delegation. A completed subagent turn simply ends, and the subagent stays reachable: a message to it resumes it from its earlier context.

## Tooling

Spawn with the `Agent` tool, and select the worker with `subagent_type`:

- Where the configuration defines agents, in the `agents/` folder of the user's Claude config directory (`~/.claude/agents/` by default) or in the project's `.claude/agents/`, choose among those by what the work needs. A definition carries the role's rules, its tools, and usually its model.
- Where none are defined, use the built-in types: `Explore` for read-only fan-out search across many files or naming conventions (it reads excerpts, so it locates code; it does not audit it), `Plan` for implementation strategy on a defined task, and `general-purpose` for everything else.

Launch independent workstreams as multiple `Agent` calls in a single message so they run concurrently. Never delegate a search and also run it yourself.

By default a subagent started from the main conversation runs in the background, and its result arrives as a notification when it stops. Go on with work that does not depend on it. Do not poll it or sleep while waiting, and never fabricate or predict a pending agent's result. A subagent's final report is not shown to the user, so relay what matters.

Use `isolation: "worktree"` only when parallel agents mutate the same files and would otherwise conflict. It costs setup time and disk per agent.

## Cost Selection

Delegated work should run at the cheapest setting that still produces a reliable result.

The `Agent` call accepts `model` and `effort`, and an agent definition can pin a model of its own.

- Pass no `model` when dispatching an agent the configuration defines. Its definition has made the choice: the model it pins, or the session's model when it pins none. Changing the tier means changing the definition, which is the user's call.
- Pass `model` only for a built-in type, and only when confident a different tier fits: a cheaper tier for source search, bounded inspection, simple audits, command execution, formatting and documentation consistency checks, mechanical edits, and focused verification; a stronger tier for non-trivial implementation judgment, multi-file causal debugging, or careful review, or when a cheaper pass came back incomplete, inconsistent, or unreliable.
- Pass `effort` only when the user or the project's instructions ask for a specific effort for that work.

If a cheaper tier appears to be reducing quality, say so in main-thread status so this preference can be revisited.

## When To Propose Subagents

Propose subagents when a task is bounded, independently verifiable, and benefits from isolated context. This applies to regular turns as well as long-running goal-mode work.

Good uses:

- inspect a bounded code area
- trace how a feature is implemented
- summarize API or UI coverage from source files
- implement one clearly scoped module, page, route, or test
- reproduce a specific bug from logs or steps
- review a diff
- run bounded verification or tool-specific checks
- perform mechanical cleanup after the direction is chosen
- compare docs against implementation
- forward-test a skill on a realistic task

Avoid subagents for:

- product vision
- ambiguous UX or architecture direction
- final acceptance decisions
- broad tasks without clear boundaries
- work requiring hidden context only the main thread has

## Approval Rule

The agent may propose subagents for bounded code inspection, implementation, review, or research.

Delegate bounded work without asking for extra approval unless a skip condition or a higher-risk approval case applies. This skill is the user's standing delegation preference; see the Precedence section in `SKILL.md` for how it ranks against Claude Code's shipped "do not delegate unless asked" baseline and against in-session user instructions.

Always ask for explicit user approval before delegating work that may:

- touch production systems
- use secrets or credentials
- perform destructive operations
- alter deployment exposure
- run expensive or long-running jobs
- make irreversible external changes

## Main Thread First

The main thread should gather the initial user intent, constraints, source artifacts, and decision context before delegating.

Delegate narrow tasks with enough context to execute. Do not send a subagent on broad discovery unless the expected result is concrete source-derived knowledge.

## Responsibility Split

Main thread owns:

- understanding the user's intent and the product
- raising architecture, UX, scope, and tradeoff decisions with the user
- task decomposition
- accepting each result, and confirming that the results together meet the request
- final verification
- user-facing status

Subagents may own:

- bounded file inspection
- source-derived summaries
- localized implementation
- targeted debugging
- test additions
- independent review findings

## The Brief

A subagent has none of your conversation. Beyond its role definition and the project's instructions, it knows what the brief says. Write the brief in three parts, in this order:

1. **Must.** Numbered requirements and nothing else, each one checkable by running a command or opening a named artifact: what has to be true when the work is done, and the verification to run.
2. **Context.** Everything the subagent needs in order to meet the Musts:
   - its own name and scope, who else is working, and what each of them owns;
   - what it must not touch, and whether edits are allowed at all;
   - the paths, logs, or diffs to work from;
   - for implementation, the change map the `coding-practices` skill defines;
   - the skills to load, named explicitly: `coding-practices` for any code change, `git-command` for commits. A subagent can invoke a skill through the Skill tool, but only reliably does so when the brief names it. An agent definition can also preload a skill through its `skills` field.
3. **Report.** What to send back: an answer to each Must by its number, the evidence for it, what could not be checked, and what was ambiguous.

Prefer raw artifacts over conclusions. Do not leak the expected answer unless the subagent is implementing an already-decided change: a subagent handed your hypothesis tends to return it confirmed.

One brief carries one chunk of work. Change a brief by sending it again whole, never by a message that amends the earlier one.

## Brief Examples

Implementation, in full:

```text
## Must
1. The Runtime Clients page shows a loading, an empty, and an error state, built from the state components the project already has.
2. `npm run check` passes.

## Context
You are `clients-ui`, the implementer that owns `src/clients/`. Nobody else is editing the frontend. Do not touch `src/auth/`, `src/navigation/`, or `deploy/`.
Change map: the clients feature owns the three states; it may import the shared state components and nothing from another feature; no boundary changes; the states are tested through the page component.
Load the `coding-practices` skill before any change.

## Report
Answer each Must by its number, with the output of the check. List each file you changed and its role.
```

What the other kinds of work ask for:

```text
Inspection: List the backend operations in the API contract that have no visible UI coverage in the frontend pages, grouped by workflow. Read only.

Review: Review this diff under two lenses: correctness, and readability and structure. Findings first, ordered by severity, each with file and line. The change map the code was built to is attached.

Skill forward-test: Use the skill at <path> to perform this task: <task>. Treat it like a normal user request. Do not review the skill directly. Produce the resulting artifact and note any ambiguity.
```

## Integration Rule

After a subagent returns:

1. Read the result critically. A report is the subagent's account of its own work, not proof.
2. Verify at least one requirement yourself, by running the command or opening the artifact the report names, and every claim the next step depends on.
3. Review structure, constraints, and project fit for delegated implementation.
4. Send a defect back to the subagent that built it, with the evidence.
5. Send the next task in the same scope to the same subagent.
6. Confirm that the accepted work, taken together, satisfies the user's request, using delegated verification where appropriate. A fix found at this step goes back to the owner of the files.

Do not present subagent output as final truth without validation. Passing checks confirms executability; it does not confirm maintainability.

## Parallelism

Use parallel subagents only when tasks are independent.

Good parallel split:

- one agent audits API coverage
- one agent audits UI coverage
- one agent reviews docs

Bad parallel split:

- multiple agents editing the same files
- agents making conflicting architecture decisions
- agents changing deployment configuration independently
