# MeGaNeKo AI Workflows

Personal agent workflow skills for Claude Code: coding practices, API contracts, Git hygiene, frontend delivery, and agent operating workflow.

Environment-specific skills, such as Docker context rules, stay local and are not bundled here.

## Layout

```text
.claude-plugin/marketplace.json          Claude Code discovery and skill selection
plugins/meganeko-ai-workflows/
  skills/                                all five skills
```

### Skills

| Skill | Scope |
| --- | --- |
| `coding-practices` | Implementation architecture, ownership, dependency seams, testing strategy. Routes to area, language, framework, and protocol references. |
| `api` | HTTP/REST contracts, OpenAPI, RFC 9457 `application/problem+json`. |
| `git-command` | Staging, commits, conventional subjects, history edits, safety. |
| `frontend-delivery` | Frontend research, planning, implementation, and runtime verification under an evidence contract. |
| `agent-workflow` | Goal mode, delegation, and context hygiene, written for Claude Code primitives. |

## Install

```powershell
claude plugin marketplace add MeGaNeKoS/claude-personal-workflows
claude plugin install meganeko-ai-workflows@meganeko-workflows
```

Or from a local clone:

```powershell
claude plugin marketplace add .
claude plugin install meganeko-ai-workflows@meganeko-workflows
```

Update with `claude plugin marketplace update meganeko-workflows` followed by `claude plugin update meganeko-ai-workflows@meganeko-workflows`, then restart. The marketplace command alone refreshes metadata but does not rebuild the installed plugin. The manifest omits `version`, so every commit is treated as a new version; uncommitted changes never install.

Validate before pushing:

```powershell
claude plugin validate .
```

## Maintenance Invariant

The "non-trivial" scope test is intentionally duplicated word for word in `plugins/meganeko-ai-workflows/skills/coding-practices/SKILL.md` and `plugins/meganeko-ai-workflows/skills/frontend-delivery/SKILL.md`. If one changes, change both.
