---
name: kanban-team-review
description: Set up and run a multi-profile Kanban pipeline — cost-optimised for AI agents (PM+Arch parallel, spec-gate, auto-merge, parallel QA).
version: 2.0.0
platforms: [linux]
metadata:
  hermes:
    tags: [kanban, multi-agent, pipeline, cost-optimised]
---

# Kanban Team Pipeline (Cost-Optimised)

AI-optimised pipeline that mirrors the quality of a human team while eliminating human-shaped bottlenecks. PM and Architect work in parallel. QA tests branches, not merged code. Clean PRs auto-merge without senior review.

## The Pipeline

```
[PM + Architect] parallel → Spec-gate → [Engineer + QA] parallel → Auto-merge → DevOps
```

### What's been cut (and why)

| Old (human-shaped) | New (AI-optimised) | Saving |
|---------------------|-------------------|--------|
| PM → Architect staged handoff | PM + Architect parallel | No PM finish-before-architect wait |
| Senior reviews every PR | Auto-merge for clean PRs, senior handles exceptions only | Pro tokens saved on 80% of PRs |
| QA after merge | QA on branches, parallel with engineers | No merge-wait-for-QA bottleneck |
| Engineer → Senior (merge) → QA | Engineer → QA parallel → Auto-merge | Merge gate eliminated |

### Spec-gate rules

Before an engineer task is dispatched, the spec must be validated:
1. PM: spec is unambiguous with acceptance criteria
2. Architect: no conflicts with ADRs or system boundaries
3. Both must have completed their work on the feature

A task blocks at spec-gate until BOTH PM and Architect are done. This prevents the #1 token waster: building the wrong thing.

### Auto-merge rules

A PR auto-merges if ALL:
- **CI is green** (GitHub Actions: `cargo check` + `cargo test` + `cargo clippy -- -D warnings`)
- No file touched is in a protected list (main.rs, lib.rs, Cargo.toml)

A PR is FLAGGED for senior review if ANY:
- CI fails (build, test, lint)
- Protected file modified
- Merge conflict detected
- Architectural drift detected

**CRITICAL:** The senior engineer must verify CI status before merging. Claims of "tests pass" without CI verification are forbidden. This caused the orchestration merge cascade failure — master broke because code was merged with red CI (no CI existed at the time). CI is now at `.github/workflows/ci.yml`.

## Process Safeguards (lessons learned Aug 09, 2026)

These are baked into every profile SOUL. Do not bypass them.

| Safeguard | Owner | Rule |
|-----------|-------|------|
| CI verification | Senior Engineer | No merge without green CI |
| Pre-flight check | DevOps | `cargo check` before `docker build` |
| Master health check | PM + Scrum Master | `cargo check` before promoting deploy |
| Self-escalation | QA + Engineers | Same failure 5× → STOP, escalate, block self |
| Single fix worker | PM + Scrum Master | One engineer per overlapping file |
| Retry loop detection | PM + Scrum Master | 2+ runs same error → unblock, reassign |
| Branch isolation | Engineers | Work on branch, push green, never merge |
| Pre-sprint health | Scrum Master | Master compiles, no retry loops, CI exists before Planning |

### Parallel QA

QA tests engineer branches immediately — does NOT wait for merge. QA creates bug tasks for issues found. Engineer fixes bugs. When both QA is green AND CI is green, auto-merge fires.

## Setup

```bash
# Profiles (use deepseek):
#   architect: pro
#   senior-engineer: pro
#   software-engineer: flash
#   qa-engineer: flash
#   product-manager: flash
#   devops: flash

# Workspace config
for p in senior-engineer software-engineer-2 software-engineer-3 qa-engineer; do
  hermes -p "$p" config set kanban.workspace_kind worktree
  hermes -p "$p" config set kanban.workspace_path /home/agent/repos/archlint-stack
done

for p in product-manager architect software-engineer devops; do
  hermes -p "$p" config set kanban.workspace_kind scratch
done

# Kanban config
hermes config set kanban.failure_limit 3
```

## Task Pipeline Pattern

```bash
# 1. PM + Architect in parallel (both get the same brief)
T_PM=$(hermes kanban create "Define <feature> — requirements and acceptance criteria" \
  --assignee product-manager --workspace scratch --body "$(cat /tmp/task-pm.md)")

T_AR=$(hermes kanban create "Design <feature> — architecture and integration plan" \
  --assignee architect --workspace scratch --body "$(cat /tmp/task-architect.md)")

# 2. Engineers depend on BOTH PM + Architect (spec-gate)
T_E1=$(hermes kanban create "Implement <feature> backend" \
  --assignee software-engineer --parent "$T_PM" --parent "$T_AR" \
  --workspace "worktree:/home/agent/repos/archlint-stack" --body "..." \
  --comment "Push green CI to auto-merge. Do NOT block with review-required.")

T_E2=$(hermes kanban create "Implement <feature> frontend" \
  --assignee software-engineer-2 --parent "$T_PM" --parent "$T_AR" \
  --workspace "worktree:/home/agent/repos/archlint-stack" --body "..." \
  --comment "Push green CI to auto-merge. Do NOT block with review-required.")

# 3. QA tests branches in parallel with engineers
#    QA depends on ANY engineer being done (not all — test branches individually)
T_QA=$(hermes kanban create "QA: Test <feature> branches" \
  --assignee qa-engineer --parent "$T_E1" --parent "$T_E2" \
  --workspace "worktree:/home/agent/repos/archlint-stack" --body "..." \
  --comment "Test engineer branches directly. Find bugs → create Kanban tasks. Don't fix bugs yourself.")

# 4. DevOps deploys after QA (no merge gate — auto-merge handled it)
T_DEPLOY=$(hermes kanban create "Deploy <feature> to K8s" \
  --assignee devops --parent "$T_QA" \
  --workspace "worktree:/home/agent/repos/archlint-stack" --body "...")
```

## Monitoring

```bash
hermes kanban list          # Board overview
hermes kanban stats         # Per-status + per-assignee counts
hermes kanban dispatch      # Force dispatcher tick
```

## Pitfalls

- **Board switching:** Always verify with `hermes kanban boards list` before `kanban create`. The current board is marked with `●`.
- **Running tasks:** Never move running tasks between boards. Workers write to their original DB.
- **`--parent` not `--parents`:** Singular, repeatable.
- **Worktree path:** Must be full path: `worktree:/home/agent/repos/archlint-stack`. Bare `worktree` causes worker spawn failures.
- **DevOps depends on QA, not merge:** No merge task exists in the new pipeline. QA is the final gate.
- **Engineer completion:** Always add comment "Push green CI to auto-merge. Do NOT block with review-required."
- **Spec-gate enforcement:** Before dispatching, run `hermes kanban show <task>` for each engineer task — verify BOTH PM and Architect parents are `done`.
- **Auto-merge setup:** Auto-merge works when CI runs automatically on push. If your repo doesn't have CI, the senior engineer handles the merge manually for all PRs (fallback).
