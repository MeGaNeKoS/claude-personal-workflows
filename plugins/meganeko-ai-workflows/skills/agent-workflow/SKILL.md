---
name: agent-workflow
description: "Use when work will span many turns or needs durable objective tracking, when deciding whether to delegate bounded work to subagents, when a thread is drifting or carrying stale context, or when choosing between asking the user and proceeding. Covers goal mode, delegation boundaries, context hygiene, and skill forward-testing."
---

# Agent Workflow

Use this skill as the single global entry point for agent operating workflow guidance.

Keep this file small. Load only the reference that matches the task.

> Claude Code variant. This file and its references name Claude Code primitives directly. The Codex plugin ships its own variant of this skill; when the judgment changes, change both.

## Base Rules

- The user owns what is built: intent, architecture, acceptance criteria, scope, and dependencies. Raise a decision about any of them with the user; do not make it.
- The main agent owns how the work is run: decomposition, briefs, checking and accepting results, and status.
- Treat goal mode as an explicit operating contract with the user. Do not start it unless the user explicitly asks for it or approves the proposal.
- Treat subagents as delegated workers or evaluators for both normal turns and goal mode, not final decision makers.
- Preserve the main thread for judgment work.
- Propose stronger workflow modes when they would materially improve the work.
- Read source artifacts before making project-specific claims.

## Precedence

Claude Code ships a baseline of "do not call the Agent tool unless the user requested it." That baseline is for users who have expressed no preference. This skill is the expressed preference and replaces it.

Order of authority, highest first:

1. An explicit instruction from the user in the current session.
2. The user's and the project's instructions: `CLAUDE.md`, `.claude/rules/*.md`, and any operating policy appended to the session's system prompt.
3. This skill.
4. Claude Code shipped defaults.

Where those instructions define roles, staffing, briefs, or ownership, they decide, and this skill fills in only what they leave open.

The higher-risk approval cases in `references/subagents.md` still require asking, regardless of the above.

## Reference Routing

Read only the branch that applies:

- Goal mode, long-running objectives, durable status, user acceptance criteria, completion/blocker rules: `references/goal-mode.md`
- Subagents, parallel agent work, model selection, briefs, checking a report, skill forward-testing: `references/subagents.md`
- Context hygiene, compaction timing, drift recovery, and preservation instructions: `references/context-hygiene.md`

Read `references/context-hygiene.md` for long-running work, resumed work, goal mode, a topic shift, or subagent-heavy work.

If a task involves multiple workflow areas, read each matching reference. Use goal-mode guidance for overall ownership, subagent guidance only for delegation, and context-hygiene guidance only for compaction decisions.

## Out Of Scope

Skill authoring, hooks, settings, and permissions are Claude Code configuration, not agent workflow. Use the `update-config` skill for those.
