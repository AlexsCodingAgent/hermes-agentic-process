# Hermes Agentic Process

Autonomous multi-agent software pipeline built on [Hermes Agent](https://hermes-agent.nousresearch.com). 10 specialized AI profiles run a Kanban pipeline — defining features, designing architectures, implementing code, reviewing, testing, and deploying — with zero human intervention in the critical path.

## Structure

```
hermes-agentic-process/
├── docs/
│   └── kanban-system-progression.md   # Full evolution: single assistant → 10-profile pipeline
├── profiles/                          # SOUL.md personality definitions for each role
│   ├── product-manager.md
│   ├── architect.md
│   ├── senior-engineer.md
│   ├── software-engineer.md
│   ├── qa-engineer.md
│   ├── devops.md
│   ├── project-manager.md
│   └── research-analyst.md
├── skills/
│   └── kanban-team-review.md          # Pipeline setup and task creation patterns
└── README.md
```

## The Pipeline (v2.0 — Cost-Optimised)

```
[PM + Architect] parallel → Spec-gate → [Engineers + QA] parallel → Auto-merge → DevOps
```

## Profiles (10 total)

| Role | Model | Tier | Purpose |
|------|-------|------|---------|
| architect | deepseek-v4-pro | PRO | System design, ADRs, API contracts |
| senior-engineer | deepseek-v4-pro | PRO | Exception-only code review, auto-merge gate |
| software-engineer ×3 | deepseek-v4-flash | FLASH | Implementation from validated specs |
| product-manager | deepseek-v4-flash | FLASH | Feature definition, prioritisation |
| qa-engineer | deepseek-v4-flash | FLASH | Branch-level testing, bug reporting |
| devops | deepseek-v4-flash | FLASH | K8s deployment, CI/CD, monitoring |
| project-manager | deepseek-v4-flash | FLASH | Pipeline monitoring, unblocking |
| research-analyst | deepseek-v4-flash | FLASH | Deep-dive research, competitor analysis |

## Key Design Decisions

- **Pro on gates, flash on work** — architect and senior-review are quality checkpoints. Everything else runs on flash.
- **Spec-gate** — no engineer starts until the spec passes an unambiguous + architect-aligned check. Prevents the #1 token waster.
- **Auto-merge** — clean CI (build, test, lint green, no protected files touched) auto-merges. Senior reviews exceptions only.
- **Parallel QA** — QA tests branches directly, not merged code. Bugs found when fixes are cheapest.
- **Separate boards per workstream** — 7 Kanban boards prevent context pollution.
- **Engineers complete, don't block** — `review-required` is a human pattern. AI pushes green and lets automation decide.

## See Also

- [kanban-system-progression.md](docs/kanban-system-progression.md) — Full 6-phase evolution history with pipeline diagrams, problems discovered, and lessons learned.
- [Hermes Agent docs](https://hermes-agent.nousresearch.com/docs)
