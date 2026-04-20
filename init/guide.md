# 新项目初始化指南

## 前置条件

- [lark-cli](https://github.com/larksuite/lark-cli) — `npm install -g @larksuite/cli`
- Obsidian（可选，用于可视化编辑 Vault）
- Git

## 步骤

### 1. 创建项目目录

```bash
mkdir my-project-vault
cd my-project-vault
git init
```

### 2. 复制 Vault 骨架

从本项目的 `init/vault-skeleton/` 复制所有目录到项目根目录：

```bash
cp -r init/vault-skeleton/* /path/to/my-project-vault/
```

或使用 `/project-manager init` 自动生成。

生成的目录结构：

```
my-project-vault/
├── 0-Projects/Daily/        # 项目看板、每日任务
├── 1-Goals/                 # OKR 目标
├── 2-Reports/Weekly/        # 周报/月报
├── 3-Meetings/              # 会议记录
├── 4-Risks/                 # 风险记录
├── 5-Team/                  # 团队成员
├── 6-Templates/             # 模板
├── 7-Archive/               # 归档
├── 8-Retrospective/         # 复盘记录
├── Meetings/Video/          # 妙记产物
└── .claude/skills/          # SKILL 占位
```

### 3. 配置飞书 token

```bash
cp templates/.env.example /path/to/my-project-vault/.env
# 编辑 .env，填入实际值
```

必填项：
| 变量 | 来源 |
|------|------|
| `FEISHU_SPACE_ID` | 飞书知识库 → 空间设置 |
| `FEISHU_WIKI_TOKEN` | 飞书知识库 → 根节点 URL |
| `BITABLE_BASE_TOKEN` | 飞书多维表格 URL |
| `BITABLE_BASE_URL` | 飞书多维表格完整 URL |
| `BITABLE_PROJECT_TABLE_ID` | 项目表 ID |
| `BITABLE_TASK_TABLE_ID` | 任务表 ID |
| `BITABLE_DAILY_TABLE_ID` | Daily 表 ID |
| `BITABLE_DASHBOARD_ID` | 仪表盘 ID |
| `FEISHU_GROUP_CHAT_ID` | 群聊 URL 中的 `oc_xxx` |

### 4. 配置团队成员

```bash
cp templates/team-registry.example.json /path/to/my-project-vault/.claude/team-registry.json
# 编辑，填入成员 open_id 和群聊 ID
```

获取 `open_id`：使用 `lark-cli` 通讯录查询，或从飞书管理后台获取。

### 5. 安装 SKILL

```bash
# 项目级别（仅当前项目）
cp -r skills/* /path/to/my-project-vault/.claude/skills/

# 或全局级别（所有项目）
cp -r skills/* ~/.claude/skills/
```

### 6. 初始化同步状态

首次运行 `/sync-init` 或 `/sync-full` 会自动创建 `.claude/sync-state.yaml`。

也可手动复制模板：

```bash
cp templates/sync-state.example.yaml /path/to/my-project-vault/.claude/sync-state.yaml
```

### 7. 验证

```bash
cd /path/to/my-project-vault

# 检查 lark-cli 认证
lark-cli auth list

# 检查配置
cat .env
cat .claude/team-registry.json

# 运行 SKILL
/project-manager        # 查看项目看板
/meeting-sync           # 同步妙记
/sync-status            # 查看同步状态
```

## 常见问题

### 找不到 `.env` 文件

确保 `.env` 在 Vault 根目录（与 `0-Projects/` 同级），而非 `.claude/` 下。

### `lark-cli auth list` 显示未登录

```bash
lark-cli auth login
```

### 业务数据目录不进入 git

骨架中已包含 `.gitignore`，排除所有 `0-Projects/` ~ `8-Retrospective/`、`Meetings/` 和 `.claude/` 配置文件。只有 SKILL 代码和模板进入版本控制。

### 多个项目共享同一套 SKILL

推荐全局安装：

```bash
cp -r skills/* ~/.claude/skills/
```

每个项目只需维护自己的 `.env` + `team-registry.json` + `sync-state.yaml`。
