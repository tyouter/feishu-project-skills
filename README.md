# Feishu Project Skills

飞书项目管理 SKILL 集合 — 与业务数据解耦，token 安全存储。

## Quick Start

1. Clone this repo
2. In your Claude session: `/setup`
3. Follow the guided prompts

That's it. All skills are installed and configured.

## Available Skills

| Skill | Trigger | Function |
|-------|---------|----------|
| `setup` | `/setup` | One-prompt guided setup wizard |
| `meeting-sync` | `/meeting-sync` | 妙记 → structured minutes → Kanban/Bitable → group notify |
| `feishu-sync` | `/sync-push`, `/sync-pull`, `/sync-full` | Vault ↔ Feishu bidirectional sync |
| `project-manager` | `/project-manager` | Dashboard, OKR, reports, risks, daily tasks |
| `lark-shared` | auto-loaded | lark-cli health check, auth, permissions |

## Prerequisites

- **lark-cli** — Feishu CLI (`npm install -g @larksuite/cli`)
- Git

## Directory Structure

```
feishu-project-skills/
├── README.md
├── skills/                     # Pure SKILL code (no hardcoded tokens)
│   ├── setup/SKILL.md          # Guided setup wizard
│   ├── meeting-sync/SKILL.md
│   ├── feishu-sync/SKILL.md
│   ├── project-manager/SKILL.md
│   └── lark-shared/SKILL.md
├── templates/                  # Config templates (empty values)
│   ├── .env.example
│   ├── team-registry.example.json
│   ├── sync-state.example.yaml
│   ├── feishu-local.example.yaml
│   └── managed-projects.example.yaml
├── init/
│   ├── guide.md               # Manual setup guide (advanced)
│   └── vault-skeleton/        # Obsidian vault directory skeleton
└── .gitignore
```

## Security

- **All tokens in `.env`**, never committed to git
- **SKILL code has no hardcoded secrets**
- **`.env` excluded in .gitignore**

## Manual Setup

If you prefer manual setup, see `init/guide.md`.
