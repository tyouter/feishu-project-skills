# Feishu Project Skills

飞书项目管理 SKILL 集合 — 与业务数据解耦，token 安全存储。

## 安装

将 `skills/` 目录复制到目标项目的 `.claude/skills/` 或全局 `~/.claude/skills/`：

```bash
# 项目级别
cp -r skills/* /path/to/project/.claude/skills/

# 或全局级别
cp -r skills/* ~/.claude/skills/
```

## 依赖

- **lark-cli** — 飞书 CLI（`npm install -g @larksuite/cli`）
- **.env** — 飞书/Bitable token 配置（见 `templates/.env.example`）
- **sync-state.yaml** — 同步状态映射（见 `templates/sync-state.example.yaml`）
- **team-registry.json** — 团队成员 open_ids（见 `templates/team-registry.example.json`）

## 可用 SKILL

| Skill | 触发方式 | 功能 |
|-------|---------|------|
| `meeting-sync` | `/meeting-sync` | 妙记自动同步 → 结构化纪要 → Kanban/Bitable 任务 → 群通知 |
| `feishu-sync` | `/sync-push`, `/sync-pull`, `/sync-full` | Vault ↔ 飞书双向同步 |
| `project-manager` | `/project-manager` | 项目看板、OKR、报告、风险、每日任务 |
| `lark-shared` | 自动加载 | 飞书 CLI 健康检查、认证、权限管理 |

## 新项目初始化

1. 复制 `init/vault-skeleton/` 到项目根目录
2. 复制 `templates/.env.example` → `.env`，填写实际值
3. 复制 `templates/team-registry.example.json` → `.claude/team-registry.json`
4. 复制 `templates/sync-state.example.yaml` → `.claude/sync-state.yaml`
5. 安装 SKILL（见上方安装说明）
6. 运行 `/meeting-sync` 或 `/project-manager` 开始使用

详见 `init/guide.md`

## 目录结构

```
feishu-project-skills/
├── README.md
├── skills/                     # 纯净 SKILL 代码（无硬编码 token）
│   ├── meeting-sync/SKILL.md
│   ├── feishu-sync/SKILL.md
│   ├── project-manager/SKILL.md
│   └── lark-shared/SKILL.md
├── templates/                  # 配置模板（空值占位）
│   ├── .env.example
│   ├── team-registry.example.json
│   └── sync-state.example.yaml
├── init/
│   └── guide.md               # 新项目初始化指南
└── vault-skeleton/             # Obsidian 仓库骨架
    ├── 0-Projects/Daily/
    ├── 1-Goals/
    ├── 2-Reports/Weekly/
    ├── 3-Meetings/
    ├── 4-Risks/
    ├── 5-Team/
    ├── 6-Templates/
    ├── 7-Archive/
    ├── 8-Retrospective/
    │   ├── Agent_Evolution/
    │   ├── Decisions/
    │   └── Lessons_Learned/
    └── Meetings/Video/
```

## 安全

- **所有 token 存储在 `.env`**，不进入 git
- **SKILL 代码无硬编码密钥**，通过 `.env` + 配置文件引用
- **`.env` 已在 .gitignore 中排除**
