# Claude-Codex Pipeline Plugin

A Claude Code plugin that combines Claude for planning and review with Codex for implementation and execution. It turns feature ideas into PRDs, user stories, tracked tasks, isolated worktrees, reviewed code, and gated merges.

## Why This Exists

AI coding assistants are powerful but chaotic. They mix planning with implementation, skip reviews, forget acceptance criteria, and produce work that's hard to trace back to requirements. Scaling to multi-file features makes it worse — changes collide, context gets lost, and there's no audit trail.

Claude-Codex Pipeline solves this by enforcing structure:

- **Two models, strict roles.** Claude thinks. Codex builds. Neither does the other's job. This separation means better plans and better code.
- **Artifact-driven handoffs.** Models communicate through files on disk (YAML specs, PRDs, acceptance criteria) — not chat transcripts. Every decision is traceable, reviewable, and version-controlled.
- **Parallel without conflicts.** Large features split into isolated git worktrees with strict file ownership. Multiple agents build simultaneously without stepping on each other.
- **Deterministic quality gates.** Lint, typecheck, tests, format — pass or fail, no model judgment. Code ships when gates pass, not when an AI says "looks good."
- **You stay in control.** Every phase pauses for your approval. You describe what you want and approve what gets built. The pipeline handles everything in between.

## What It Does

```
You describe a feature
  → Claude writes a PRD
  → Claude splits it into user stories
  → Stories sync to Linear
  → Stories group into parallel tasks
  → Each task gets an isolated git worktree
  → Codex agents build each task simultaneously
  → Gates validate every task (lint, typecheck, tests, format)
  → Claude reviews the diffs
  → Branches merge to main in safe order
  → Linear issues close
```

Every step produces a file. Every file is in your repo. Nothing is hidden in chat history.

## Three Ways to Use It

### Quick: Ship a single task

```
/claude-codex-pipeline:ship "add rate limiting to the API"
```

Claude plans → Codex implements → Claude reviews → Codex fixes → gates pass → done. One branch, one loop. Good for bug fixes, focused features, refactors.

### Medium: Delegate parallel work

```
/claude-codex-pipeline:delegate "redesign the authentication system"
```

Claude decomposes the work into independent tasks. Each gets its own worktree with strict file boundaries. Agents build in parallel. Merge in safe order when done.

### Full: End-to-end feature pipeline

```
/claude-codex-pipeline:feature-flow "add OAuth login with Google and GitHub"
```

The complete lifecycle — idea to shipped code — with approval gates between every phase:

```
1. Intake     → capture the idea
2. PRD        → structured requirements doc          ← you approve
3. Stories    → atomic units of work (YAML files)    ← you approve
4. Tracker    → push to Linear
5. Tasks      → group stories by file ownership      ← you approve
6. Build      → parallel Codex agents in worktrees
7. Validate   → deterministic gate checks
8. Merge      → safe order, rebase-first             ← you approve
```

## Key Values

### Traceability

Every decision lives in a file. The PRD traces to stories. Stories trace to tasks. Tasks trace to branches. Branches trace to diffs. Reviews trace to findings with file and line number. Months later, you can read `docs/features/FEATURE-001/` and understand exactly what was built, why, and how it was validated.

### Safety at Scale

Path ownership prevents collisions. Task A cannot touch Task B's files — enforced by hooks. Merge order is deterministic: read-only changes first, shared config last. Gates re-run after every rebase. No merge happens without passing gates.

### Human Authority

The pipeline never auto-merges (unless you pass `--auto-merge`). It never skips review. It never ships without gates passing. You approve the plan, the stories, the task breakdown, and the merge. The AI does the work; you make the decisions.

### Reproducibility

No magic. The pipeline is YAML files, git worktrees, and deterministic checks. If something breaks, read the artifacts. If you disagree with a plan, edit the YAML. If you want to re-run a phase, the artifacts are right there. Nothing depends on chat context or model memory.

## Getting Started

### 1. Install the plugin

```bash
claude plugin install claude-codex-pipeline@8lianno
```

### 2. Install dependencies

You also need [Codex CLI](https://github.com/openai/codex) and the codex-toolkit plugin:

```bash
npm install -g @openai/codex
claude plugin install codex-toolkit@xiaolai
```

Optional — for independent review gating:

```bash
claude plugin install review-loop@hamel-review
```

### 4. Scaffold your repo

```
/claude-codex-pipeline:init-swarm
```

This creates: `CLAUDE.md`, `AGENTS.md`, `TASK_OWNERS.yaml`, `swarm-policy.yaml`, `.claude/settings.json`, and the `docs/` directory structure. Optionally connects Linear for issue tracking.

### 5. Build something

```
/claude-codex-pipeline:ship "add input validation to the signup form"
```

Or go bigger:

```
/claude-codex-pipeline:feature-flow "implement Stripe billing with usage-based pricing"
```

## Architecture

```
Claude plans → Codex implements → Claude reviews → Codex fixes → gate passes → done
```

| Role | Model | What It Does |
|------|-------|-------------|
| Planner | Claude | Writes PRDs, decomposes features, reviews diffs |
| Builder | Codex | Implements code, runs tests, fixes review blockers |
| Gatekeeper | Deterministic | Lint, typecheck, tests, format — no AI judgment |
| Decision-maker | You | Approves plans, stories, merges |

Models never talk to each other directly. They communicate through artifacts on disk:
- `docs/tasks/TASK-NNN.yaml` — what to build
- `docs/acceptance/TASK-NNN.yaml` — how to validate
- `docs/review/REVIEW-TASK-NNN.yaml` — what to fix

## Commands

| Command | Purpose |
|---------|---------|
| `/claude-codex-pipeline:init-swarm` | Scaffold repo for first use |
| `/claude-codex-pipeline:feature-flow <idea>` | Full pipeline: idea to shipped code |
| `/claude-codex-pipeline:ship <task>` | Single task: plan → build → review → gate |
| `/claude-codex-pipeline:delegate <feature>` | Split into parallel worktree tasks |
| `/claude-codex-pipeline:merge-tasks` | Merge completed task branches |
| `/claude-codex-pipeline:sync-tracker` | Push/pull/status with Linear |
| `/claude-codex-pipeline:swarm-status` | Overview of all tasks and worktrees |

## Artifact Structure

```
docs/
  features/FEATURE-001/
    intake.yaml              ← the original idea
    prd.md                   ← requirements doc
    stories/
      STORY-001.yaml         ← unit of work
      STORY-002.yaml
      index.yaml             ← story summary
    tracker-map.yaml         ← Linear sync bridge
  tasks/
    TASK-001.yaml            ← execution spec
  acceptance/
    TASK-001.yaml            ← gate config
  review/
    REVIEW-TASK-001.yaml     ← review findings
TASK_OWNERS.yaml             ← path ownership registry
```

## Dependencies

| Dependency | Purpose |
|------------|---------|
| [Claude Code](https://claude.com/claude-code) | CLI runtime |
| [Codex CLI](https://github.com/openai/codex) | Implementation engine |
| `codex-toolkit` plugin | Orchestrated Codex workflows |
| Linear (optional) | Issue tracking via native MCP |

## License

MIT
