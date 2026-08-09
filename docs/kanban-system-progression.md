# Hermes Kanban — System Progression

From a single default AI assistant to a 10-profile, 7-board autonomous software pipeline.

---

## Phase 1: Single Assistant (baseline)

**Before the team existed.**

Alex used Hermes Agent directly — one profile, one model, all tasks. The agent wrote code, debugged, deployed, and managed everything sequentially. No delegation, no pipeline, no specialization.

```
[Alex] → [default profile] → does everything
```

**Limitations:** No parallelism. No quality gate. Every bug was fixed by the same entity that wrote it. No separation between "what should we build" and "how should we build it."

---

## Phase 2: Five-Profile Pipeline (v1.0)

**The first team.** Introduced the human-shaped software lifecycle pattern.

Profiles created:
- `product-manager` — defines features, writes specs
- `architect` — designs systems, produces ADRs
- `senior-engineer` — code review, technical gaps
- `software-engineer` — implements features
- `qa-engineer` — tests and validates

Model tiers: `deepseek-v4-pro` for architect and senior-engineer (quality gates), `deepseek-v4-flash` for PM, engineer, and QA (cost-optimized).

Pipeline:
```
PM → Architect → Senior (review) → Engineer → Senior (merge) → QA
```

Single `default` Kanban board. Senior engineer appeared twice — once as code reviewer (parallel with PM), once as merge gate.

**Problems discovered:**
- PM bottlenecked on research tasks — long-running research blocked all feature definition
- `review-required` pattern: engineers completed work but blocked for review instead of marking done
- Single board crowding: all workstreams mixed together
- Workspace path bug: bare `worktree` caused spawn failures
- DevOps dependency chain errors: devops linked to merge instead of QA

---

## Phase 3: Specialization + Capacity (v1.1)

**Splitting roles for parallel throughput.**

New profiles:
- `research-analyst` — deep-dive research (offloaded from PM)
- `project-manager` — automated pipeline monitoring and unblocking (became `afb819dd2a23` cron job, every 5 minutes)
- `software-engineer-2`, `software-engineer-3` — parallel implementation workers
- `devops` — K8s deployment, Cloudflare tunnels, image builds

10 profiles total:
```
pro:  architect, senior-engineer
flash: product-manager, research-analyst, project-manager,
       software-engineer, software-engineer-2, software-engineer-3,
       qa-engineer, devops
```

Full pipeline with merge gate:
```
PM ──────────────┐
                  ├──→ Architect ──┐
Senior (review) ──┘                ├──→ Engineers (×3) ──→ Merge ──→ QA ──→ DevOps
```

Kanban config: `failure_limit=3`, engineer tasks instructed to `complete` not `block`, DevOps parent must be QA (not merge).

---

## Phase 4: Multi-Board Workstreams (v1.2)

**Separating concerns across boards.**

Created per-workstream boards. Tasks moved from default board to dedicated boards via SQLite copy + archive on source:

| Board | Workstream | Status |
|-------|-----------|--------|
| `default` | Operational/ad-hoc | 5 tasks |
| `reporting-dashboard` | Reporting dashboard feature | 18 tasks (live) |
| `browser-cache` | Browser caching improvements | 6 tasks (done) |
| `archlint-slidev` | Slidev integration | 9 tasks (done) |
| `training-scenarios` | Training scenarios system | 20 tasks |
| `orchestration-ui` | Admin panel + team management | 9 tasks (building) |
| `archlint` | Platform feature requests | 1 task |

---

## Phase 5: Reporting Dashboard (v1.3)

**Real-time visibility into the pipeline.**

Built and deployed at `http://192.168.1.145:30089/reporting` (internal-only, WireGuard VPN access from phone).

Features:
- Pipeline view with dependency arrows and color-coded task cards
- Board selector (switch between all boards)
- Worker panel (all 10 profiles, model tier, current task, recent history)
- Blocked panel with root cause display and unblock actions
- Health badge (green/yellow/red based on blocked count)
- Auto-refresh every 30 seconds
- Mobile-responsive (dark theme, 44px+ touch targets)
- Data synced via `kubectl cp` cron every 5 minutes (no SSH dependency)

Deployment: Rust/Axum backend serves static HTML, reads kanban.db in WAL mode, NodePort 30089.

---

## Phase 6: Cost-Optimised Pipeline (v2.0)

**From human-shaped to AI-shaped.**

Identified that the pipeline mirrored human team constraints — and AI doesn't have them.

Changes:
1. **PM + Architect parallel** — both start from the same brief. Co-design loop replaces staged handoff.
2. **Spec-gate** — task must pass an unambiguous spec check before an engineer touches it. Prevents the #1 token waster: building the wrong thing.
3. **Auto-merge** — clean PRs (build + test + lint green, no protected files) auto-merge. Senior engineer reviews exceptions only.
4. **Parallel QA** — QA tests engineer branches directly, not merged code. Bugs found at branch level when fixes are cheapest.

Pipeline before (v1):
```
PM → Architect → Senior → [3 Engineers] → Merge → QA → DevOps
4 sequential handoffs, 2 pro reviews
```

Pipeline after (v2):
```
[PM + Architect] parallel → Spec-gate → [Engineers + QA] parallel → Auto-merge → DevOps
1 handoff, 0 staged pro reviews
```

Estimated token savings: 30-40% — primarily from eliminating mechanical senior review and preventing spec-driven rework.

---

## Current State Summary

| Component | Count |
|-----------|-------|
| Profiles | 10 (2 pro, 8 flash) |
| Boards | 7 |
| Total tasks | ~67 across all boards |
| Running workers | Typically 3-5 simultaneously |
| Cron jobs | 2 (project-manager 5min check, kubectl cp 5min sync) |
| Deployment | K8s (hermes namespace, Iota control plane) |
| Access | Internal NodePort 30089 via WireGuard VPN |

## Key Lessons

1. **Separate boards per workstream** — single `default` board doesn't scale past 3 features. Seven boards keep context clean.
2. **Engineers complete, don't block** — `review-required` is a human pattern. AI should push green and let automation decide.
3. **Pro on gates, flash on work** — architect and senior-review are your quality checkpoints. Everything else can run on flash without quality loss.
4. **Board switching is fragile** — always verify `hermes kanban boards list` shows the right `●` before creating tasks. Moving tasks between boards is SQLite surgery.
5. **Running tasks can't move between boards** — workers write to their original DB. If you move a running task, it becomes a zombie on the new board.
6. **SSH isn't guaranteed** — kubectl cp is a reliable cross-node file transfer when SSH key distribution is uneven.
