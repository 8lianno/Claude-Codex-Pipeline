# Project Rules

## Architecture: Claude + Codex Pipeline

This project uses a two-model pipeline. Claude owns planning and review. Codex owns implementation and execution. Never let both write to the same branch simultaneously.

### The Flow

1. **Claude plans** (read-only) → produces `docs/plan.md` + `docs/acceptance.yaml`
2. **Codex implements** against the plan artifact (not the transcript) → via `/implement`
3. **Claude reviews** the diff + test report → produces `docs/review.md`
4. **Codex fixes** from review blockers → via `/continue` or `/implement`
5. **Deterministic gate** before done → lint, typecheck, tests, format

### Command Cheatsheet

| Phase | Command | Who |
|-------|---------|-----|
| Plan | Enter plan mode or `/ship` | Claude |
| Implement | `/implement docs/plan.md` | Codex via MCP |
| Audit | `/audit [scope]` or `/audit-fix [scope]` | Codex via MCP |
| Review plan | `/review-plan docs/plan.md` | Codex via MCP |
| Review loop | `/review-loop <task>` | Claude impl + Codex review |
| Verify fixes | `/verify` | Codex via MCP |
| Continue thread | `/continue <threadId>` | Codex via MCP |
| Full pipeline | `/ship <task>` | Orchestrated |

### Handoff Rules

- Handoffs happen through artifacts: `docs/plan.md`, `docs/acceptance.yaml`, `docs/review.md`
- Never pass raw chat history between models
- One writer at a time on any branch
- Always include acceptance criteria before implementation
- Always run deterministic checks before declaring done

### Planning Output Contract

When writing `docs/plan.md`, include:
- Problem statement
- Files/modules likely to change
- Constraints and non-goals
- Step-by-step implementation plan
- Acceptance tests (also in `docs/acceptance.yaml`)
- Rollback notes

### Implementation Output Contract

After Codex implements, verify:
- Changed files list
- Commands run
- Test report
- Unresolved issues

### Review Contract

When reviewing a diff, check:
- Architecture fit with existing code
- Missed edge cases
- Regression risk
- Product correctness against acceptance criteria
- Security (OWASP top 10)

## Parallel Work: Worktree Delegation

For multi-task features, use `/delegate` to split work across isolated worktrees.

### Core Principle

One task = one branch = one worktree = one active writer. The main checkout is the supervisor — no direct coding here.

### Path Ownership

- Every task declares allowed paths and denied paths in `TASK_OWNERS.yaml`
- If a task needs a file owned by another task, it STOPS and reports an ownership conflict
- Shared files (lockfiles, root config, CI, schema) are supervisor-owned — edited only after all tasks merge
- `TASK_OWNERS.yaml` is the single source of truth for who owns what

### Commands

| Action | Command |
|--------|---------|
| Split & delegate | `/delegate <feature description>` |
| Merge completed tasks | `/merge-tasks [task-name \| all]` |
| List worktrees | `git worktree list` |
| Single-task pipeline | `/ship <task>` (within a worktree) |

### Merge Order

1. Read-only analysis tasks (priority 1)
2. Leaf module changes (priority 2)
3. Test-only changes (priority 3)
4. Shared/core/config changes last (priority 4)

Always rebase onto main before merging. Run full gate checks post-rebase.

### Agent Assignment

- **Codex**: Pure implementation, mechanical changes, test writing
- **Claude**: Design-heavy, ambiguous, requires codebase understanding
- **Either as reviewer**: Read-only review in any worktree (never co-editing live)

### Worktree Layout

```
repo/                          # supervisor — planning and coordination only
  .claude/worktrees/
    task-a/                    # worktree for task A
    task-b/                    # worktree for task B
  docs/
    delegation/
      task-a.md                # task A spec
      task-b.md                # task B spec
    acceptance/
      task-a.yaml              # task A criteria
      task-b.yaml              # task B criteria
  TASK_OWNERS.yaml             # path ownership registry
```

## General Rules

- Do not over-engineer. Only make changes that are directly requested.
- Prefer editing existing files over creating new ones.
- Run tests after every implementation pass.
- Keep `docs/` artifacts up to date throughout the pipeline.
