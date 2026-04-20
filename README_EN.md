# Feishu Project Management Skills

AI-powered project management toolkit built on Claude Code + lark-cli, bridging Feishu (Lark) and Obsidian vault.

中文版：[README.md](./README.md)

---

## Quick Start

1. Clone this repo
2. Type `/setup` in your Claude session
3. Follow the guided prompts

That's it. All skills are installed and configured.

---

## Key Features

### Auto Meeting Minutes Sync

- Auto-discovers Feishu meeting minutes, deduplicates, generates structured minutes
- Extracts decisions, action items, task completion status
- Auto-maps to local Kanban and Bitable tasks
- Group notifications with @mentions and summary

### Auto Daily TODO Generation

- AI breaks down Kanban tasks into day-level sub-tasks
- Assigns daily tasks to team members with multi-person collaboration support
- Dual-write: local MD file + Feishu Bitable real-time sync
- Auto-scans incomplete tasks, reschedules, and re-evaluates breakdowns
- Historical daily task records available

### CLI Bot for Auto Project Management

- `/project-manager` — Dashboard: OKR, projects, kanban, risks at a glance
- `/project-manager kanban sync` — Local Kanban ↔ Feishu Bitable bidirectional sync
- `/project-manager check-deadlines` — Deadline alerts (overdue / within 3 days / this week)
- `/project-manager follow-up` — Auto-generates follow-up reminders
- `/project-manager velocity` — Team productivity tracking + trend analysis
- `/project-manager parse-meeting` — Meeting note parsing: 5 categories (decisions / actions / risks / assignments / dependencies)

### Vault ↔ Feishu Bidirectional Sync

- Local Obsidian MD ↔ Feishu Wiki bidirectional sync
- Conflict detection: timestamp comparison, user chooses strategy (push / pull / abort / diff)
- Automatic conflict backup to `.claude/conflicts/`
- Per-file precise sync with `/sync-file <path>`
- YAML frontmatter auto-stripped (prevents Feishu rendering issues)

### Retrospective Mechanism

- **Agent Evolution** — AI Agent changes auto-recorded (triggers on SKILL.md modification)
- **Lessons Learned** — Experience library, prompts to record on task completion
- **Decisions (ADR)** — Key decision records, auto-numbered `DEC-{date}-{seq}`
- **Sprint Retro** — Auto-generated Sprint retrospective report (completion rate / velocity / overdue items / recommendations)

### Risk Management

- Risk intake, review, status tracking
- Auto-identifies risk signals from meetings ("有风险" / "来不及" / "搞不定")
- Severity-based sorting, auto-marks stale risks

### Kanban Board Management

- `0-Projects/Kanban.md` as single source of truth, bidirectional sync with Bitable
- Project/task status mapping (not started / in progress / completed / cancelled)
- Auto-deliverable verification (code commit / doc update / meeting notes / tests)
- Multi-person task splitting, independent Bitable record per person

### Team Velocity Tracking

- Records estimated vs actual days per task
- Weekly velocity trend visualization
- Underestimate pattern recognition (e.g., tasks involving external communication typically 1.5x slower)
- Sprint capacity recommendations

### Git Safety

- Managed projects are **READ-ONLY**: no commit/push/reset/checkout
- Vault itself can commit/push
- `git pull --ff-only` before any operation to stay in sync
- Auto-stash + backup on conflict, no destructive operations

---

## Available Skills

| Skill | Trigger | Function |
|-------|---------|----------|
| `setup` | `/setup` | Guided one-prompt setup wizard |
| `project-manager` | `/project-manager` | Full project lifecycle management |
| `meeting-sync` | `/meeting-sync` | Minutes → structured → task sync |
| `feishu-sync` | `/sync-push`, `/sync-pull`, `/sync-full` | Bidirectional vault sync |
| `lark-shared` | auto-loaded | CLI health check, auth, permissions |

---

## Prerequisites

- **lark-cli** — Feishu CLI (`npm install -g @larksuite/cli`)
- **Claude Code** — AI runtime
- **Git** — Version control

---

## Directory Structure

```
feishu-project-skills/
├── README.md                    # 中文版
├── README_EN.md                 # English version
├── skills/                      # Pure SKILL code (no hardcoded tokens)
│   ├── setup/SKILL.md           # Guided setup wizard
│   ├── meeting-sync/SKILL.md    # Auto meeting minutes sync
│   ├── feishu-sync/SKILL.md     # Bidirectional vault sync
│   ├── project-manager/SKILL.md # Project management core
│   └── lark-shared/SKILL.md     # lark-cli shared rules
├── templates/                   # Config templates (empty values)
│   ├── .env.example             # Feishu/Bitable tokens
│   ├── team-registry.example.json  # Team member open_ids
│   ├── sync-state.example.yaml  # Sync state mappings
│   ├── feishu-local.example.yaml    # Personal override config
│   └── managed-projects.example.yaml  # Managed projects registry
├── init/
│   ├── guide.md                # Manual setup guide (advanced)
│   └── vault-skeleton/         # Obsidian vault directory skeleton
└── .gitignore
```

---

## Config Cascade

All skills use the same unified config loading order:

| Priority | File | Purpose |
|----------|------|---------|
| 1 | `.env` | Feishu/Bitable tokens |
| 2 | `.claude/team-registry.json` | Team member open_ids + group chats |
| 3 | `.claude/sync-state.yaml` | Sync mappings, statistics |
| 4 | `.claude/feishu-local.yaml` | Personal overrides (optional) |

---

## Security

- **All tokens in `.env`**, never committed to git
- **SKILL code has no hardcoded secrets**
- **`.env` excluded in .gitignore**
- Sensitive configs (sync-state.yaml, team-registry.json, etc.) excluded from version control

---

## Manual Setup

For manual setup (debugging or custom scenarios), see `init/guide.md`.
