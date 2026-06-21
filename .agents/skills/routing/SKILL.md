---
name: routing
description: "Use when starting any conversation - resolves which skills to use in this project. Project-local workstream skills replace legacy solopowers equivalents."
---

# Project Skill Routing

This project uses Workstream-Driven Development. The following global skills are **superseded** and must NOT be invoked:

| Superseded global skill | Replaced by (project-local) |
|-------------------------|----------------------------|
| `brainstorming` | `workstream-brainstorming` |
| `writing-plans` | _(eliminated — workstream doc replaces separate plans)_ |
| `executing-plans` | `workstream-driven-development` |
| `subagent-driven-development` | `workstream-driven-development` |

## Routing Rules

**Feature work, new functionality, or multi-file changes:**
→ `workstream-brainstorming` → `workstream-driven-development`

**Bug fixes, single-file changes, refactors that don't need design:**
→ Work directly. Use `systematic-debugging`, `tdd`, `verification-before-completion` as appropriate.

**Code review (requesting or receiving):**
→ Use `requesting-code-review` / `receiving-code-review` as-is.

**Branch setup and completion:**
→ Use `using-git-worktrees` and `finishing-a-development-branch` as-is.

## Skills that remain active (no conflict)

- `using-git-worktrees`
- `finishing-a-development-branch`
- `verification-before-completion`
- `systematic-debugging`
- `requesting-code-review`
- `receiving-code-review`
- `tdd` / `test-driven-development`
- `frontend-design`
- `find-skills`
- `dispatching-parallel-agents` (for non-workstream parallel tasks)
