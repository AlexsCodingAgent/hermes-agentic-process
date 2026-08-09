# Kanban System — Team Structure & Board Analysis

State as of August 09, 2026. Autonomous multi-agent software pipeline on Hermes Agent with DeepSeek models.

---

## Team Structure

### Active Profiles (10)

| # | Profile | Model | Tier | Role | Throughput |
|---|---------|-------|------|------|-----------|
| 1 | `product-manager` | deepseek-v4-flash | FLASH | Feature definition, requirements, prioritisation | done=1 |
| 2 | `architect` | deepseek-v4-pro | PRO | System design, ADRs, API contracts | done=4 |
| 3 | `senior-engineer` | deepseek-v4-pro | PRO | Exception-only code review, merge gate | done=2 |
| 4 | `software-engineer` | deepseek-v4-flash | FLASH | Implementation from validated specs | done=1 |
| 5 | `software-engineer-2` | deepseek-v4-flash | FLASH | Parallel implementation worker | done=1 |
| 6 | `software-engineer-3` | deepseek-v4-flash | FLASH | Parallel implementation worker | done=1 |
| 7 | `qa-engineer` | deepseek-v4-flash | FLASH | Branch-level testing, bug reporting | running=1 |
| 8 | `devops` | deepseek-v4-flash | FLASH | K8s deployment, CI/CD, infrastructure | done=1, todo=1 |
| 9 | `project-manager` | deepseek-v4-flash | FLASH | Pipeline monitoring, unblocking (cron 5min) | idle |
| 10 | `research-analyst` | deepseek-v4-flash | FLASH | Deep-dive research, competitor analysis | idle |

### Dormant / Legacy Profiles (7)

| Profile | Status |
|---------|--------|
| `default` | Original profile — idle |
| `archlint` | Unused — idle |
| `euicc` | Other project — idle |
| `iot-careers` | Other project — idle |
| `ops` | Unused — idle |
| `ghost-profile` | No SOUL on disk — used for UI tests |
| `qa-worker` | No SOUL on disk — used for e2e tests |

### Concurrent Workers

Typical: **3-4** workers running simultaneously. Peak: 5-6 during parallel implementation phases. Idle when no tasks have satisfied dependencies.

---

## Pipeline (v2.0 — Cost-Optimised)

```
[PM + Architect] parallel → Spec-gate → [Engineers + QA] parallel → Auto-merge → DevOps
```

### Key Design Rules

| Rule | Reason |
|------|--------|
| PM + Architect work in parallel | Same brief, different concerns. Co-design loop replaces staged handoff |
| Spec-gate before engineer dispatch | Task must pass unambiguous + architect-aligned check. Prevents building wrong thing |
| Engineers complete, don't block | Auto-merge handles clean PRs. No `review-required` pattern |
| Auto-merge on green CI | Build + test + lint + no protected files → merge automatically |
| Senior reviews exceptions only | Flagged only for CI failures, merge conflicts, protected file changes |
| QA tests branches, not merged code | Parallel with engineers. Bugs caught when fixes are cheapest |
| DevOps depends on QA | Never on merge. Chain: … → QA → DevOps |

### Model Allocation

Pro tokens (expensive, high-quality): `architect`, `senior-engineer`

Flash tokens (cheap, fast): everything else

Rationale: pro on quality gates only. Flash handles implementation, testing, deployment.

---

## Boards

7 boards, 80 active tasks. 73 done, 1 running, 1 todo.

### Board Summary

| Board | Total | Done | Running | Blocked | Todo | Profiles | Workstream |
|-------|-------|------|---------|---------|------|----------|-----------|
| `default` | 7 | 7 | 0 | 0 | 0 | 1 | Operational tasks, system hygiene |
| `browser-cache` | 6 | 6 | 0 | 0 | 0 | 1 | Browser caching improvements |
| `archlint-slidev` | 9 | 9 | 0 | 0 | 0 | 1 | Slidev presentation integration |
| `training-scenarios` | 22 | 22 | 0 | 0 | 0 | 1 | Architecture training system (18→30 scenarios) |
| `reporting-dashboard` | 18 | 18 | 0 | 0 | 0 | 1 | Live pipeline visibility dashboard |
| `orchestration-ui` | 16 | 14 | 1 | 0 | 1 | 3 | Admin panel + team management |
| `archlint` | 2 | 2 | 0 | 0 | 0 | 1 | Platform feature requests |

### Board Details

#### default (7 tasks, all done)
```
  ✓ t_5a2e70cf  Fix: /practise generic surface submit button stays disabled
  ✓ t_c99396d9  Fix: /practise generic surface drafts not autosaved/restored
  ✓ t_c027e636  Review and merge cache invalidation fix
  ✓ t_029ec557  Fix: project list cache invalidation broken
  ✓ t_35cffc38  QA re-verify cache invalidation fix
  ✓ t_d2265f3e  Align k8s manifests to hermes namespace
  ✓ t_3bc09a7c  Research grading methodologies
```

#### browser-cache (6 tasks, all done)
```
  ✓ PM → Senior (review) → 3 Engineers (parallel) → Merge → QA → DevOps
```
Complete pipeline. Browser caching layers implemented, tested, deployed.

#### archlint-slidev (9 tasks, all done)
```
  ✓ PM → Senior → Architect → Engineer → QA → Merge → DevOps → P0 fixes → Deploy
```
Slidev integration complete. Server.js, K8s image refs, base path all fixed.

#### training-scenarios (22 tasks, all done)
```
  ✓ PM (18 scenarios) → Senior (review) → Architect → 3 Engineers → Merge → QA → DevOps
  ✓ Bug fix chain: 5 fixes → merge → QA → deploy
  ✓ Expansion: 18→30 scenarios
  ✓ /practise route fix chain: fix → merge → QA
```
Largest workstream. 6-phase architecture training system with timer, scenario picker, auto-grading. Survived multiple bug-fix cycles on the /practise page.

#### reporting-dashboard (18 tasks, all done)
```
  ✓ PM → Senior → Architect → 3 Engineers → Merge → QA → DevOps
  ✓ Bug fix: 503/404 error path
  ✓ Mobile UX: header collapse, stats scroll, bottom nav, pull-to-refresh
  ✓ Mobile polish: collapsible workers, bottom sheet drawer
  ✓ Fix chain: 3 viewport bugs → done
```
Live at `http://192.168.1.145:30089/reporting` (internal, WireGuard VPN access). Pipeline view, worker panel, blocked panel, health badge, board selector. Data synced via kubectl cp cron every 5 minutes.

#### orchestration-ui (16 tasks: 14 done, 1 running, 1 todo)
```
  ✓ PM → Architect → Senior → 3 Engineers → Merge
  ● QA (running) → DevOps (todo)
```
Last board still active. Multi-board dashboard, team management page, orchestration actions (dispatch, complete, unblock, reassign, create/archive boards). Waiting on QA sign-off before deploy.

#### archlint (2 tasks, both done)
```
  ✓ t_981306e9  Implement sandbox side-by-side code+preview
  ✓ t_5797e72a  Sandbox: side-by-side code + preview (feature request)
```
Platform feature requests. Sandbox split-pane implemented.

---

## Active Work

| Board | What's happening |
|-------|-----------------|
| `orchestration-ui` | QA testing admin panel. DevOps deploys after sign-off |
| 5 other boards | All tasks complete. Pipe dry |

---

## Infrastructure

| Component | Detail |
|-----------|--------|
| **Deployment** | K8s (hermes namespace, Iota control plane 192.168.1.145) |
| **Registry** | 192.168.1.145:30500 (internal) |
| **Public access** | Cloudflare tunnel → archlint.tuddenham.dev |
| **Internal access** | NodePort 30089 → reporting dashboard (WireGuard VPN) |
| **Database** | SQLite WAL mode, per-board kanban.db files |
| **Board sync** | kubectl cp cron every 5 minutes (no SSH dependency) |
| **Pipeline monitor** | project-manager cron every 5 minutes |
| **Kanban config** | failure_limit=3, notification_sources=[*] |

---

## Patterns Observed

### Proven to Work
1. **PM + Architect parallel** — both start from a single-line brief. Zero rework from spec/architecture mismatch
2. **3 parallel engineers** on independent files — no conflicts, clean merges every time
3. **Spec-gate implicit** — PM and Architect being done before engineer starts prevents wrong builds
4. **QA finding bugs → creating tasks → engineers fixing** — proper separation of concerns
5. **Per-workstream boards** — context stays clean. 80 tasks across 7 boards is manageable

### Friction Points
1. **Board switching is unreliable** — tasks sometimes land on wrong board. Verify with `kanban boards list` before creating
2. **Running tasks can't move boards** — workers write to original DB. Moving creates zombies
3. **DevOps deploy runs hit security denials** — Docker builds and SSH connections get blocked by the scanner
4. **SSH key distribution uneven** — kubectl cp is a reliable fallback for cross-node file transfer
5. **Merge gate pattern (old)** replaced by auto-merge — but not all repos have CI to support it

---

## Cost Profile

| Worker type | Est. cost per task | Active frequency |
|-------------|-------------------|-----------------|
| Pro (architect, senior) | ~$0.10-0.20 | 1-2 tasks per feature |
| Flash (all others) | ~$0.01-0.03 | 4-8 tasks per feature |

**Idle profiles cost zero.** Only running workers consume tokens. The dispatcher spawns workers only for ready tasks with satisfied dependencies.
