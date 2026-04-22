# Feishu Project Skills

本仓库是 Claude Code + lark-cli 的 AI 自动化项目管理工具集。克隆后运行 `/setup` 即可完成初始化。

## Skills

本仓库提供以下 skills（位于 `.claude/skills/`）：

- **setup** (`/setup`) — 一键引导配置向导，创建 vault 骨架、分发 skills、配置 .env
- **project-manager** (`/project-manager`) — 项目全生命周期管理：仪表盘、OKR、看板、风险、每日任务、效能追踪
- **meeting-sync** (`/meeting-sync`) — 飞书妙记自动发现 → 结构化纪要 → 任务同步 → 群通知
- **feishu-sync** (`/sync-push`, `/sync-pull`, `/sync-full`, `/sync-file`, `/sync-status`, `/sync-init`) — Obsidian ↔ 飞书知识库双向同步
- **lark-shared** (自动加载) — lark-cli 健康检查、认证、权限、安全规则

## 前置条件

- **lark-cli** — `npm install -g @larksuite/cli`
- **Claude Code** — AI 运行时
- **Git** — 版本控制

## 配置级联

所有 Skills 统一使用同一套配置加载顺序（目标 vault 中）：

| 优先级 | 文件 | 用途 |
|--------|------|------|
| 1 | `.env` | 飞书/Bitable Token |
| 2 | `.claude/team-registry.json` | 团队成员 open_ids + 群聊 |
| 3 | `.claude/sync-state.yaml` | 同步映射、统计数据 |
| 4 | `.claude/feishu-local.yaml` | 个人覆盖（可选） |

## 快速开始

1. 克隆本仓库
2. 在 Claude Code 对话中输入 `/setup`
3. 按引导提示完成配置（vault 骨架 + .env + 团队注册 + lark-cli 健康检查）

## 安全

- 所有 Token 存储在目标 vault 的 `.env`，不提交到 git
- SKILL 代码无硬编码密钥
- 受管项目（managed projects）只读：禁止 commit/push/reset/checkout
