# Manual Setup Guide (Advanced)

> **Note:** Most users should use `/setup` instead. This guide is for manual setup, debugging, or custom configurations.

## Prerequisites

- [lark-cli](https://github.com/larksuite/lark-cli) — `npm install -g @larksuite/cli`
- Obsidian (optional, for visual vault editing)
- Git

## Steps

### 1. Create Project Directory

```bash
mkdir my-project-vault
cd my-project-vault
git init
```

### 2. Copy Vault Skeleton

Copy all directories from `init/vault-skeleton/` to the project root:

```bash
cp -r init/vault-skeleton/* /path/to/my-project-vault/
```

Generated structure:

```
my-project-vault/
├── 0-Projects/Daily/        # Project kanban, daily tasks
├── 1-Goals/                 # OKR goals
├── 2-Reports/Weekly/        # Weekly/monthly reports
├── 2-Reports/Monthly/
├── 3-Meetings/              # Meeting notes
├── 4-Risks/                 # Risk records
├── 5-Team/                  # Team member profiles
├── 6-Templates/             # Templates
├── 7-Archive/               # Archive
├── 8-Retrospective/         # Retrospective records
│   ├── Agent_Evolution/
│   ├── Decisions/
│   └── Lessons_Learned/
├── Meetings/Video/          # Meeting video minutes
└── .claude/skills/          # SKILL placeholder
```

### 3. Configure Feishu Tokens

```bash
cp templates/.env.example /path/to/my-project-vault/.env
# Edit .env with actual values
```

Required tokens:
| Variable | Source |
|----------|--------|
| `FEISHU_SPACE_ID` | 飞书知识库 → 空间设置 |
| `FEISHU_WIKI_TOKEN` | 飞书知识库 → 根节点 URL last segment |
| `BITABLE_BASE_TOKEN` | 飞书多维表格 URL after /base/ |
| `BITABLE_BASE_URL` | 飞书多维表格 full domain |
| `BITABLE_PROJECT_TABLE_ID` | 多维表格项目表 ID |
| `BITABLE_TASK_TABLE_ID` | 多维表格任务表 ID |
| `BITABLE_DAILY_TABLE_ID` | 多维表格每日任务表 ID |
| `BITABLE_DASHBOARD_ID` | 多维表格仪表盘 ID |
| `FEISHU_GROUP_CHAT_ID` | 飞书群聊 URL `oc_xxx` part |

### 4. Configure Team Members

```bash
cp templates/team-registry.example.json /path/to/my-project-vault/.claude/team-registry.json
# Edit with actual member open_ids and group chat ID
```

Get `open_id`: use `lark-cli contact user +search --query "<name>"` or from Feishu admin console.

### 5. Install Skills

```bash
# Project-level (this project only)
cp -r .claude/skills/* /path/to/my-project-vault/.claude/skills/

# Or global-level (all projects)
cp -r .claude/skills/* ~/.claude/skills/
```

### 6. Initialize Sync State

`/sync-init` or `/sync-full` auto-creates `.claude/sync-state.yaml`.

Or copy template manually:

```bash
cp templates/sync-state.example.yaml /path/to/my-project-vault/.claude/sync-state.yaml
```

### 7. Verify

```bash
cd /path/to/my-project-vault

# Check lark-cli auth
lark-cli auth list

# Check config
cat .env
cat .claude/team-registry.json

# Run skills
/project-manager        # View project dashboard
/meeting-sync           # Sync meeting minutes
/sync-status            # View sync status
```

## Common Issues

### Missing `.env` file

Ensure `.env` is at vault root (same level as `0-Projects/`), NOT inside `.claude/`.

### `lark-cli auth list` shows not logged in

```bash
lark-cli auth login
```

### Business data not in git

The vault skeleton `.gitignore` excludes `0-Projects/` through `8-Retrospective/`, `Meetings/`, and `.claude/` config files. Only SKILL code and templates are tracked in git.

### Multiple projects share same skills

Install globally:

```bash
cp -r .claude/skills/* ~/.claude/skills/
```

Each project only needs its own `.env` + `team-registry.json` + `sync-state.yaml`.
