# Scrum Implementation Plan for Hermes Agent Pipeline

Plan to layer Scrum sprint cycles on top of the existing continuous-flow Kanban system.

---

## 1. Theoretical Scrum Team Structure

### Standard Scrum Roles vs AI Adaptation

| Role | Human Version | AI Agent Adaptation |
|------|-------------|-------------------|
| **Product Owner (PO)** | One person. Owns product backlog. Prioritizes. Accepts/rejects work. | `product-manager` (existing). Already does this. Needs sprint commitment authority. |
| **Scrum Master (SM)** | Facilitator. Removes impediments. Runs ceremonies. Protects team from interruption. | **NEW**: `scrum-master` profile. Proactive process owner. Runs cron-driven ceremonies. |
| **Development Team** | 3-9 cross-functional people. Self-organizing. No sub-teams. | 7 existing profiles: architect, senior-engineer, 3× software-engineer, qa-engineer, devops. Cross-functional by design. |

### Why not separate PO and SM?

Human teams keep PO and SM separate to avoid conflicts (PO pushes for more, SM protects team velocity). AI agents don't have political dynamics. But separating them still matters:

- PO focuses on **what** (backlog, priorities, stakeholder value)
- SM focuses on **how well the process runs** (ceremonies, blockers, metrics)

Same one person can do both in small teams. For AI, separating them makes each profile's SOUL cleaner and more effective.

### Team Size

Scrum recommends 3-9 on the development team. Current team: 7 (architect, senior, 3 engineers, QA, DevOps). Fits perfectly.

---

## 2. Sprint Cycle Design

### Sprint Length

Human teams: 2 weeks. AI agents work ~20x faster on focused work.

| Sprint length | Task throughput | Ceremony overhead | Best for |
|--------------|----------------|-------------------|---------|
| 6 hours | 2-3 features | High (50% ceremony) | Urgent work |
| **24 hours** | 5-8 features | Medium (20% ceremony) | **Recommended** — natural daily rhythm |
| 3 days | 12-20 features | Low (5% ceremony) | Long-form features |
| 1 week | 30+ features | Low | Complex systems |

**Recommendation: 24-hour sprints.** Aligns with daily cycle. Sprint planning at 09:00, review at 08:45 next day, retro at 08:30. Work happens continuously between.

### Sprint Cycle Flow

```
00:00  Sprint starts
09:00  Sprint planning (30 min)         ← PO + SM + Dev Team
       - PO presents top backlog items
       - Team estimates and commits
       - Sprint goal defined
       - Tasks created on Sprint Board

09:30  Sprint execution begins
       - Engineers pull from Sprint Backlog
       - QA tests completed work
       - Senior handles exceptions

  Every hour: Daily Scrum (5 min, cron)  ← SM generates standup report
       - What was done since last standup?
       - What will be done next?
       - Any impediments?

23:30  Sprint execution stops

23:45  Sprint Review (15 min)           ← PO + SM
       - Demo completed items
       - PO accepts/rejects
       - Unfinished → Back to Product Backlog

00:00  Sprint Retrospective (15 min)    ← SM + Dev Team
       - What went well?
       - What to improve?
       - Action items for next sprint

00:15  Backlog refinement (15 min)      ← PO
       - Update priorities
       - Split large items
       - Prepare next sprint

00:30  Rest period (system cooldown)

09:00  Next sprint planning
```

### Ceremony Time Budget (24-hour sprint)

| Ceremony | Duration | Led by | Participants | Cron |
|----------|----------|--------|-------------|------|
| Sprint Planning | 30 min | SM | PO + Dev Team | Scheduled |
| Daily Scrum | 5 min/hour | SM | Dev Team (async report) | Cron hourly |
| Sprint Review | 15 min | SM | PO + stakeholders | Scheduled |
| Sprint Retro | 15 min | SM | Dev Team | Scheduled |
| Backlog Refinement | 15 min | PO | PO solo | Follows retro |
| **Total** | ~2.5 hours | | | 10% of sprint |

---

## 3. Profiles Needed

### Existing (7, reused)

| Profile | Scrum Role | Sprint responsibility |
|---------|-----------|----------------------|
| `product-manager` | Product Owner | Backlog ownership, prioritization, sprint goal, accept/reject work |
| `architect` | Dev Team | Design, ADRs, tech decisions |
| `senior-engineer` | Dev Team | Exception review, code quality |
| `software-engineer` ×3 | Dev Team | Implementation |
| `qa-engineer` | Dev Team | Branch testing, bug reporting |
| `devops` | Dev Team | Deployment, infrastructure |

### New (1)

| Profile | Scrum Role | Sprint responsibility |
|---------|-----------|----------------------|
| **`scrum-master`** | Scrum Master | Ceremony facilitation, impediment removal, sprint board management, metrics, burndown |

### Total: 8 active profiles (down from 10 — PM and research-analyst merge into PO activities)

`project-manager` and `research-analyst` become sprint-adjacent: the project-manager cron still monitors pipeline health, but sprint management moves to SC. Research-analyst handles deep-dive tasks for PO when needed (not sprint-bound).

---

## 4. Scrum Master SOUL Design

```
Profile: scrum-master
Model: deepseek-v4-flash
Toolset: terminal, file, read, search, kanban

Core responsibilities:
- Facilitate sprint ceremonies (planning, standup, review, retro)
- Maintain sprint board (move tasks, track progress)
- Remove impediments (unblock blocked tasks, restart crashed workers)
- Generate burndown chart and velocity metrics
- Protect sprint scope (push unplanned work to next sprint)
- Coach team on process (flag when flow breaks)

Ceremony triggers (cron-driven):
- Sprint planning: read product backlog, present top N items, facilitate commitment
- Hourly standup: check sprint board, assess progress vs burndown, flag at-risk items
- Sprint review: demo completed items to PO, get accept/reject decisions
- Retrospective: analyze sprint metrics, propose process improvements

What they DON'T do:
- Define what to build (PO)
- Design architecture (architect)
- Write code (engineers)
- Test (QA)
- Deploy (DevOps)
- Set product priorities (PO)
```

---

## 5. Sprint Board Design

### New Board: `sprint-board`

Replaces the per-workstream board pattern during active sprints. One board per sprint.

```
Sprint Backlog  →  In Progress  →  In Review  →  Done
   (todo)          (running)       (ready)       (done)
```

Kanban status mapping:
- `todo` = Sprint Backlog (committed but not started)
- `running` = In Progress (engineer working)
- `done` = Done (completed by engineer, auto-validated)
- QA creates bug tasks → go to Sprint Backlog as `todo`

### Product Backlog Board: `product-backlog`

Long-lived board. Ordered by priority. Not sprint-bound.

```
Backlog (todo) → Ready for Sprint (ready) → [moves to sprint-board during planning]
```

Columns:
- `todo` = unrefined, not ready
- `ready` = refined, estimated, ready for sprint planning
- During sprint planning, PO promotes `ready` items to the sprint board

### Board Lifecycle

```
Product Backlog           Sprint Board                Archive
    (permanent)          (one per sprint)          (after sprint)
   ┌─────────┐          ┌──────────────┐          ┌──────────┐
   │ todo    │  ─────→  │ sprint backlog│  ─────→  │ done     │
   │ ready   │  plan    │ in progress  │  review  │ accepted │
   │         │          │ in review    │          │          │
   │         │          │ done         │          │ rejected │
   └─────────┘          └──────────────┘          └──────────┘
```

---

## 6. Ceremonies as Cron Jobs

### Hourly Standup (`scrum-standup`)
```
Schedule: @hourly
Action: SM checks sprint board
Output: Standup report with:
  - Tasks completed since last standup
  - Tasks currently running
  - Impediments (blocked tasks, crashed workers, stale heartbeats)
  - Burndown: completed/committed ratio vs sprint elapsed
  - Flag if sprint goal is at risk
Deliver: to origin chat (Alex gets hourly pulse)
```

### Sprint Planning (`scrum-planning`)
```
Schedule: 09:00 daily
Action: SM + PO run planning ceremony
Output:
  - PO presents backlog top N items with priorities
  - SM captures team capacity (based on previous sprint velocity)
  - Tasks committed and moved to sprint board
  - Sprint goal documented
  - Sprint board created/cleared
Deliver: sprint plan + sprint goal
```

### Sprint Review (`scrum-review`)
```
Schedule: 08:45 daily (15 min before next sprint)
Action: SM reviews sprint board, presents completed items
Output:
  - Completed vs committed
  - Demo notes for accepted items
  - Rejected/unfinished items → back to product backlog
  - Velocity metric (completed story points / sprint)
Deliver: sprint review summary
```

### Sprint Retrospective (`scrum-retro`)
```
Schedule: 08:30 daily (30 min before next sprint)
Action: SM analyzes sprint
Output:
  - What went well (metrics, patterns)
  - What to improve (bottlenecks, rework, blockers)
  - Action items for next sprint
  - Process adjustments to SOUL files if needed
Deliver: retro summary + action items
```

---

## 7. Metrics and Artifacts

### Burndown Chart
Generated by SM each standup. Tracks remaining sprint backlog vs time.

```
Tasks
 25 │╲
    │ ╲___
 20 │     ╲___
    │        ╲___
 15 │           ╲___
    │              ╲___
 10 │                 ╲___
    │                    ╲
  5 │                      ╲___
    │                         ╲
  0 └──────────────────────────────
    0h  6h   12h   18h   24h
```

### Velocity
Rolling average of completed tasks per sprint over last 3 sprints. Feeds sprint planning capacity commitment.

### Sprint Goal
One sentence. Defines what the sprint achieves. Example: "Deliver sandbox split-pane preview with draggable divider."

### Definition of Done
Per-task: code pushed, CI green, QA passed, PO accepted.

### Cumulative Flow Diagram
Tracked by SM. Shows work distribution across columns over time. Identifies bottlenecks (e.g., review queue growing = need more senior bandwidth).

---

## 8. What Changes from Current Kanban

| Aspect | Current (Continuous Kanban) | Proposed (Scrum Sprints) |
|--------|---------------------------|------------------------|
| Work intake | Continuous — tasks created as needed | Sprint-bound — only what's committed in planning |
| Planning horizon | Feature by feature | 24-hour sprint goals |
| WIP limits | Implicit (depends on concurrency) | Explicit (sprint backlog is the WIP limit) |
| Metrics | Health badge (blocked count) | Burndown, velocity, CFD, sprint goal achievement |
| Ceremonies | Project manager cron (passive observer) | SM cron (active facilitator) |
| Board structure | Per-workstream (7 boards) | Product backlog + sprint board (2 boards) |
| Roles | 10 profiles | 8 profiles (consolidated, +1 SM) |
| Cadence | None — flow-based | 24-hour sprint rhythm |
| Commitment | None | Sprint goal + committed backlog |

## 9. Which to Keep?

**Recommendation: Keep both.** Use Kanban for operational/maintenance work (the `default` board). Use Scrum sprints for feature development. Workstream boards (reporting, training, etc.) become sprint scope sources — their unstarted tasks feed the product backlog.

```
Operational (Kanban)              Feature work (Scrum)
  default board           ←→       product-backlog → sprint-board → archive
  (flow, no ceremonies)            (sprints, 24h, ceremonies)
```

---

## 10. Implementation Order

| Phase | What | Effort |
|-------|------|--------|
| 1 | Create `scrum-master` profile + SOUL | 30 min |
| 2 | Create `product-backlog` and `sprint-board` boards | 15 min |
| 3 | Implement sprint planning cron + SM prompt | 1 hour |
| 4 | Implement hourly standup cron | 45 min |
| 5 | Implement sprint review + retro crons | 1 hour |
| 6 | Migrate existing workstream tasks to product backlog | 30 min |
| 7 | Run first 24-hour sprint | 1 day |
| 8 | Retro → adjust → iterate | Ongoing |

**Total setup: ~4 hours. First sprint: 24 hours. Feedback loop: immediately.**
