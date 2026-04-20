# 飞书项目管理 Skills / Feishu Project Management Skills

基于 Claude Code + lark-cli 的 AI 自动化项目管理工具集，实现飞书与 Obsidian 知识库的双向打通。

AI-powered project management toolkit built on Claude Code + lark-cli, bridging Feishu (Lark) and Obsidian vault.

---

## 快速开始 / Quick Start

1. 克隆本仓库 / Clone this repo
2. 在 Claude 对话中输入 `/setup` / Type `/setup` in your Claude session
3. 按引导提示完成配置 / Follow the guided prompts

全部安装完成。开箱即用。

That's it. All skills are installed and configured.

---

## 核心功能 / Key Features

### 妙记自动同步 / Auto Meeting Minutes Sync

- 自动发现飞书妙记，去重后生成结构化会议纪要
- 提取决策项、行动项、任务完成状态
- 自动映射到本地 Kanban 和 Bitable 任务
- 群通知自动 @责任人 并推送摘要

Auto-discovers Feishu meeting minutes, deduplicates, generates structured minutes with decisions, action items, and task status. Auto-maps to local Kanban and Bitable tasks. Sends group notifications with @mentions.

### 每日任务自动生成 / Auto Daily TODO Generation

- 基于 Kanban 大任务自动拆解到天粒度子任务
- 按团队成员分配每日任务，支持多人协作拆分
- 双写机制：本地 MD 文件 + 飞书 Bitable 实时同步
- 自动扫描未完成任务，顺延并重新评估拆解
- 支持查看历史每日任务记录

AI breaks down Kanban tasks into day-level sub-tasks, assigns them to team members, dual-writes to local MD + Feishu Bitable. Auto-scans incomplete tasks and re-evaluates breakdowns.

### CLI 自动化项目管理 / CLI Bot for Auto Project Management

- `/project-manager` — 项目仪表盘：OKR、项目、看板、风险一览
- `/project-manager kanban sync` — 本地看板 ↔ 飞书 Bitable 双向同步
- `/project-manager check-deadlines` — 截止日期预警（过期/3天内/本周）
- `/project-manager follow-up` — 自动生成跟进提醒
- `/project-manager velocity` — 团队效能追踪 + 趋势分析
- `/project-manager parse-meeting` — 会议记录解析，提取 5 类信息（决策/行动项/风险/人员变更/依赖关系）

Dashboard, Kanban-Bitable sync, deadline checks, follow-up reminders, velocity tracking, and meeting note parsing — all from CLI commands.

### 知识库双向同步 / Vault ↔ Feishu Bidirectional Sync

- 本地 Obsidian MD ↔ 飞书知识库双向同步
- 冲突检测：时间戳比对，用户选择策略（推送/拉取/中止/对比）
- 自动备份冲突文件到 `.claude/conflicts/`
- 单文件精准同步 `/sync-file <path>`
- YAML frontmatter 自动剥离（避免飞书渲染异常）

Bidirectional sync with conflict detection, automatic backup, per-file sync, and YAML frontmatter stripping.

### 复盘机制 / Retrospective Mechanism

- **Agent Evolution** — AI Agent 变更自动记录（SKILL.md 修改自动触发）
- **Lessons Learned** — 经验教训库，任务完成时自动提醒是否记录
- **Decisions (ADR)** — 关键决策记录，自动编号 `DEC-{date}-{seq}`
- **Sprint 复盘** — 自动生成本 Sprint 复盘报告（完成率/效能/逾期项/建议）

Automated Agent change tracking, Lessons Learned prompts on task completion, ADR-style decision records, and auto-generated Sprint retrospective reports.

### 风险管理 / Risk Management

- 风险录入、评审、状态追踪
- 会议中自动识别风险信号（"有风险"/"来不及"/"搞不定"）
- 按严重程度排序，自动标记陈旧风险

Risk intake, review, tracking. Auto-identifies risk signals from meetings.

### 看板管理 / Kanban Board Management

- `0-Projects/Kanban.md` 为唯一数据源，与 Bitable 双向同步
- 项目/任务状态映射（未开始/进行中/已完成/已取消）
- 交付物自动验证（代码 commit / 文档更新 / 会议纪要 / 测试）
- 多人任务拆分，每人独立 Bitable 记录

Single-source Kanban with Bitable sync. Auto-deliverable verification. Multi-person task splitting.

### 团队效能追踪 / Team Velocity Tracking

- 记录每次任务估算天数 vs 实际天数
- 每周效能趋势可视化
- 低估模式识别（如涉及外部沟通的任务通常慢 1.5x）
- Sprint 容量推荐

Tracks estimated vs actual days, weekly velocity trends, underestimate pattern analysis, Sprint capacity recommendations.

### Git 安全策略 / Git Safety

- 受管项目（managed projects）**只读**：禁止 commit/push/reset/checkout
- Vault 自身可 commit/push
- 每次操作前 `git pull --ff-only` 确保代码同步
- 冲突时自动 stash + 备份，不自动 reset/force

Managed projects are READ-ONLY. Vault auto-pulls before any operation. No destructive git operations.

---

## 可用 Skills / Available Skills

| Skill | 触发方式 Trigger | 功能 Function |
|-------|---------|------|
| `setup` | `/setup` | 一键引导配置向导 / Guided setup wizard |
| `project-manager` | `/project-manager` | 项目全生命周期管理 / Full project lifecycle mgmt |
| `meeting-sync` | `/meeting-sync` | 妙记 → 结构化纪要 → 任务同步 / Minutes → tasks sync |
| `feishu-sync` | `/sync-push`, `/sync-pull`, `/sync-full` | 知识库双向同步 / Bidirectional vault sync |
| `lark-shared` | 自动加载 auto-loaded | CLI 健康检查、认证、权限 / Health check, auth, permissions |

---

## 前置条件 / Prerequisites

- **lark-cli** — 飞书 CLI（`npm install -g @larksuite/cli`）
- **Claude Code** — AI 运行时
- **Git** — 版本控制

---

## 目录结构 / Directory Structure

```
feishu-project-skills/
├── README.md
├── skills/                     # 纯净 SKILL 代码（无硬编码 token）
│   ├── setup/SKILL.md          # 引导式配置向导
│   ├── meeting-sync/SKILL.md   # 妙记自动同步
│   ├── feishu-sync/SKILL.md    # 知识库双向同步
│   ├── project-manager/SKILL.md# 项目管理核心
│   └── lark-shared/SKILL.md    # lark-cli 共享规则
├── templates/                  # 配置模板（空值占位）
│   ├── .env.example            # 飞书/Bitable Token
│   ├── team-registry.example.json  # 团队成员 open_id
│   ├── sync-state.example.yaml # 同步状态映射
│   ├── feishu-local.example.yaml   # 个人覆盖配置
│   └── managed-projects.example.yaml  # 受管项目注册表
├── init/
│   ├── guide.md               # 手动配置指南（高级）
│   └── vault-skeleton/        # Obsidian 仓库骨架
└── .gitignore
```

---

## 配置级联 / Config Cascade

所有 Skills 统一使用同一套配置加载顺序：

| 优先级 | 文件 | 用途 |
|--------|------|------|
| 1 | `.env` | 飞书/Bitable Token |
| 2 | `.claude/team-registry.json` | 团队成员 open_ids + 群聊 |
| 3 | `.claude/sync-state.yaml` | 同步映射、统计数据 |
| 4 | `.claude/feishu-local.yaml` | 个人覆盖（可选） |

---

## 安全 / Security

- **所有 Token 存储在 `.env`**，不提交到 git
- **SKILL 代码无硬编码密钥**，通过配置文件引用
- **`.env` 已在 `.gitignore` 中排除**
- 敏感配置（sync-state.yaml、team-registry.json 等）不进入版本控制

---

## 手动配置 / Manual Setup

如需手动配置（调试或自定义场景），详见 `init/guide.md`。
