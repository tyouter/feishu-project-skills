---
name: feishu-sync
description: Vault ↔ Feishu bidirectional sync with conflict handling. Push local to cloud, pull cloud to local, full sync.
trigger: /sync-push /sync-pull /sync-full /sync-file /sync-status /sync-init
---

# /feishu-sync

Bidirectional sync between Obsidian Vault and Feishu (Lark) Wiki with automatic conflict handling.

## Feishu Target

Read Wiki config from `.env` (tokens), `.claude/sync-mapping.yaml` (static mapping), `.claude/sync-state.yaml` (runtime state), and `.claude/team-registry.json` (team), with optional override from `.claude/feishu-local.yaml`:
- `feishu.wiki_token` → Wiki node token (from `sync-mapping.yaml` or `.env`)
- `feishu.wiki_name` → Wiki display name (from `sync-mapping.yaml`)
- `feishu.space_id` → Wiki space ID (from `sync-mapping.yaml` or `.env`)
- `bitable.*` → Bitable table config (from `sync-mapping.yaml` or `.env`)

## Usage

```
/sync-push                    # Push all local changes to Feishu
/sync-pull                    # Pull all Feishu changes to local
/sync-full                    # Full bidirectional sync + conflict handling
/sync-file <local_path>       # Sync single file
/sync-status                  # Show sync status dashboard
/sync-init                    # Initialize mapping registry
```

## Commands

### /sync-push (Local → Feishu)

Push local changes to Feishu Wiki **with conflict detection**:

1. **Read sync-state.yaml** for current mappings
2. **Scan local files** for modifications since `last_sync`
3. **For each modified file:**
   - **Step A — Detect remote changes (conflict check):**
     - Query Feishu node metadata:
       ```bash
       lark-cli wiki spaces get_node --params '{"token":"<feishu_node>"}'
       ```
     - Extract `obj_edit_time` (Unix timestamp) → convert to ISO format
     - Compare with mapping's `feishu_modified`:
       - `obj_edit_time > feishu_modified` → **Remote has changes** → **STOP and prompt user**
       - `obj_edit_time == feishu_modified` → No remote changes → proceed
     - **User prompt format:**
       ```
       ⚠️ Conflict detected: <filename>
       - Local modified:  <local_modified>
       - Feishu modified: <feishu_obj_edit_time>
       - Last sync:       <last_sync>

       Choose action:
       [1] Push local → overwrite Feishu
       [2] Pull Feishu → overwrite local
       [3] Abort (no sync)
       [4] Show diff
       ```
     - Wait for user decision before proceeding
   - **Step B — Push content (if approved):**
     - Read file content (Markdown)
     - **Strip YAML frontmatter** (the `--- ... ---` block at the top) — Feishu renders it as visible text
       ```python
       import re
       stripped = re.sub(r'^---\n.*?\n---\n', '', content, flags=re.DOTALL, count=1)
       ```
     - Write stripped content to temp file in `.claude/` directory (relative path for lark-cli)
     - **Determine parent folder:** Check `sync-state.yaml` → `folder_mappings` to find the matching `feishu_node` for the file's directory. If no mapping found, use `feishu.wiki_token` from config as fallback.
     - Check if Feishu document exists in mapping
     - If exists: `lark-cli docs +update --doc "<doc_token>" --mode overwrite --markdown @./.claude/<tempfile> --new-title "<title>"`
     - If new: Create doc directly in the correct wiki folder with title (ONE step, NOT two):
       ```bash
       lark-cli docs +create --title "<doc_title>" --markdown @./.claude/<tempfile> --wiki-node "<parent_feishu_node>" --wiki-space "<space_id>"
       ```
       **CRITICAL:**
       - Use `docs +create --wiki-node` to create directly in the wiki folder in ONE step. This sets both title and content correctly.
       - Do NOT use the old two-step flow (`docs +create` then `wiki nodes create`) — it creates a separate empty doc with no title.
       - `<parent_feishu_node>` must be the `feishu_node` from `folder_mappings` matching the file's directory (e.g., `2-Reports` → `LIHewe39Min58skUDZac8eMdn2g`), NOT the root `wiki_token`.
       - Extract `<doc_title>` from the first H1 heading (`# Title`) in the markdown content, or derive from the filename.
       - `<space_id>` is read from `.env` (`FEISHU_SPACE_ID`) or `sync-mapping.yaml` → `feishu.space_id`.
     - Clean up temp file
4. **Update sync-state.yaml** with new timestamps
5. **Update stats** (successful_push count)

**Lark CLI Commands:**
```bash
# Update existing document
lark-cli docs +update --doc "<doc_token>" --mode overwrite --markdown "$(cat file.md)" --new-title "<title>"

# Create new document directly in Wiki folder (ONE step)
lark-cli docs +create --title "<title>" --markdown "$(cat file.md)" --wiki-node "<parent_node>" --wiki-space "<space_id>"
```

### /sync-pull (Feishu → Local)

Pull Feishu changes to local vault:

1. **List Wiki node contents**
   ```bash
   lark-cli wiki nodes list --params '{"token":"<wiki_token>"}'
   ```
2. **Get metadata for each document**
   ```bash
   lark-cli drive metas batch_query --params '{"tokens":["<doc_token>"]}'
   ```
3. **Compare timestamps:**
   - `feishu_modified > last_sync` → Feishu has changes
   - `local_modified > last_sync` → Local has changes
   - Both → **Conflict detected**
4. **Handle conflicts:**
   ```bash
   git stash push -m "sync-conflict-backup-$(date '+%Y%m%d%H%M%S')"
   # Backup to .claude/conflicts/
   ```
5. **Fetch and write content:**
   ```bash
   lark-cli docs +fetch --doc "<doc_token>" --format json
   # Write to local file
   ```
6. **Update sync-state.yaml**

### /sync-full (Bidirectional)

Complete bidirectional sync with conflict handling:

1. **Execute `/sync-push`** first
2. **Execute `/sync-pull`** second
3. **Handle all conflicts** according to strategy:
   - Git stash local changes
   - Backup to `.claude/conflicts/`
   - Log conflict details to `sync-log.md`
   - Use Feishu content (cloud_overwrite strategy)
4. **Update stats** (last_full_sync timestamp)

### /sync-file <path>

Sync a single file with conflict detection:

1. **Check if file exists in mapping**
2. **Detect changes on both sides:**
   - Local: compare file's mtime vs `local_modified` in mapping
   - Feishu: `lark-cli wiki spaces get_node --params '{"token":"<feishu_node>"}'` → `obj_edit_time` vs `feishu_modified`
3. **Decision logic:**
   - Local changed, Feishu unchanged → **Push** (ask user to confirm)
   - Feishu changed, Local unchanged → **Pull** (ask user to confirm)
   - Both changed → **Conflict** → ask user to choose: push / pull / abort
   - Neither changed → skip (already in sync)

### /sync-status

Display sync dashboard:

```
## Sync Status Dashboard

### Statistics
- Total Syncs: X
- Successful Push: X
- Successful Pull: X
- Conflicts Detected: X
- Last Full Sync: YYYY-MM-DD HH:MM

### Synced Files (X)
| Local Path | Feishu Doc | Status | Last Sync |

### Pending Changes
- Local modified files: X
- Feishu modified docs: X

### Conflicts
| File | Local Modified | Feishu Modified | Backup Location |
```

### /sync-init

Initialize the mapping registry:

1. **List all Wiki node documents**
2. **Match with local files** by filename similarity
3. **Create initial mappings** with current timestamps
4. **Write to sync-state.yaml**

## Conflict Handling Flow

### Detection (Proactive — before every push/pull)

1. Query Feishu node: `lark-cli wiki spaces get_node --params '{"token":"<node>"}'`
2. Extract `obj_edit_time` (Unix timestamp) → compare with mapping's `feishu_modified`
3. Compare with mapping's `local_modified` → determine which side changed

### Resolution (User-in-the-loop)

```
┌─────────────────────────────────────────────────────┐
│              Conflict Detected                      │
├─────────────────────────────────────────────────────┤
│ 1. Query Feishu obj_edit_time → compare timestamps  │
│ 2. Detect change source:                            │
│    - Local only → auto-push (or ask confirm)        │
│    - Feishu only → auto-pull (or ask confirm)       │
│    - Both changed → ask user to choose              │
│ 3. User choices:                                    │
│    [1] Push local → overwrite Feishu                │
│    [2] Pull Feishu → overwrite local                │
│    [3] Abort (no sync)                              │
│    [4] Show diff (fetch + compare)                  │
│ 4. If overwrite chosen:                             │
│    - git stash local backup                         │
│    - Copy to .claude/conflicts/                     │
│    - Log to .claude/sync-log.md                     │
│ 5. Update mapping with new sync time                │
│                                                     │
│ User can recover from:                              │
│ - git stash list → git stash pop                    │
│ - .claude/conflicts/ backup files                   │
└─────────────────────────────────────────────────────┘
```

## Lark CLI Reference

```bash
# Wiki Operations
lark-cli wiki nodes list --params '{"token":"<wiki_token>"}'
lark-cli wiki nodes +create --parent "<wiki_token>" --obj_type docx --obj_token "<doc_token>"

# Document Operations
lark-cli docs +create --title "<title>" --markdown "$(cat file.md)" --wiki-node "<parent_node>" --wiki-space "<space_id>"
lark-cli docs +fetch --doc "<doc_token>" --format json
lark-cli docs +update --doc "<doc_token>" --mode overwrite --markdown "$(cat file.md)" --new-title "<title>"

# Drive Metadata
lark-cli drive metas batch_query --params '{"tokens":["<doc_token>"]}'

# Media Download (for images)
lark-cli docs +media-download --doc "<doc_token>" --file_token "<file_token>"
```

## File Structure

| File | Purpose |
|------|---------|
| `.claude/sync-mapping.yaml` | Static mapping: folder_mappings, file_mappings, bitable config, conflict strategy |
| `.claude/sync-state.yaml` | Runtime state: sync timestamps, stats |
| `.claude/sync-log.md` | Sync operation log |
| `.claude/conflicts/` | Conflict backup directory |

## Limitations

- **Only DocX type documents** can sync (no Whiteboard)
- **Images/media** require separate download via `+media-download`
- **Manual trigger** - no automatic sync
- **Requires Feishu app permissions** (enterprise approved)

## Integration with Git

The vault has Obsidian Git configured (10-minute auto-backup):

- Conflicts are stashed before overwrite
- Users can `git stash pop` to recover local versions
- Sync operations logged to git-trackable files

## Notes

- Use `lark-cli auth login` if authentication expired
- Check `lark-cli config show` for current app settings
- Wiki token: read from `.env` (override via `.claude/feishu-local.yaml`)

### Config Loading Cascade

Read config in priority order before any Feishu operation:

1. **`.env`** — environment variables for Feishu/Bitable tokens
2. **`.claude/team-registry.json`** — team member open_ids + group chat IDs
3. **`.claude/sync-mapping.yaml`** — static mapping: folder_mappings, file_mappings, feishu_only_docs, root_mappings, bitable config (primary source for mapping data)
4. **`.claude/sync-state.yaml`** — runtime state: sync timestamps, stats (supplementary to sync-mapping)
5. **`.claude/feishu-local.yaml`** — personal override (optional, may not exist)

Agent must check `.env` first for tokens, then `sync-mapping.yaml` for static mapping data, then `sync-state.yaml` for runtime state.

## Kanban ↔ Feishu Bitable Sync

### Bitable Configuration

Read Bitable config from `.env` (tokens), `.claude/sync-mapping.yaml` (bitable section), with optional override from `.claude/feishu-local.yaml`:
- `bitable.base_token` → Bitable base token
- `bitable.base_url` → Bitable URL
- `bitable.project_table` → Project table name
- `bitable.task_table` → Task table name
- `bitable.project_table_id` → Project table ID (from `sync-mapping.yaml`)
- `bitable.task_table_id` → Task table ID (from `sync-mapping.yaml`)
- `bitable.daily_table_id` → Daily task table ID (from `sync-mapping.yaml`)

### Sync Flow (Kanban → Bitable)

1. **Read local kanban** (`0-Projects/Kanban.md`)
2. **Parse projects and tasks** from markdown tables
3. **For each project:**
   - If not in Bitable (name match): create record
   - If exists: update status, deadline, owners if changed
   ```bash
   lark-cli base +record-create --base-token <token> --table-id "🚩 项目" --record '{"项目名称":"...","状态":"...","项目截止时间":"...","总负责人":[...]}'
   lark-cli base +record-update --base-token <token> --table-id "🚩 项目" --record-id <id> --record '{...}'
   ```
4. **For each task:**
   - If not in Bitable (name match): create record, link to project
   - If exists: update status, priority, assignee if changed
   ```bash
   lark-cli base +record-create --base-token <token> --table-id "✅ 任务" --record '{"任务":"...","状态":"...","优先级":"...","任务执行人":[...],"所属项目":["recXXX"]}'
   ```
5. **Update bidirectional links**: project ←→ task link fields
6. **Update kanban frontmatter** with `last_sync` timestamp

### Sync Flow (Bitable → Kanban)

1. **Fetch all records** from both tables:
   ```bash
   lark-cli base +record-list --base-token <token> --table-id "🚩 项目" --limit 100
   lark-cli base +record-list --base-token <token> --table-id "✅ 任务" --limit 200
   ```
2. **Parse Bitable data** into project/task structures
3. **Merge with local kanban**:
   - New Bitable records → append to Kanban
   - Changed status/priority → update local
   - Keep local-only fields (notes, descriptions)
4. **Write merged Kanban.md**
5. **Update frontmatter** `last_sync` timestamp

### Record ID Resolution

Bitable uses record IDs (e.g., `recvgVWS9Uh335`) for linking. The kanban tracks these in a hidden column:
- Tasks reference project via `所属项目` field (record ID)
- Projects reference tasks via `对应任务` field (bidirectional link)

When syncing, resolve by name → ID mapping:
1. Build name → ID map from both tables
2. Use display names for matching, record IDs for API calls

### Conflict Resolution

| Scenario | Resolution |
|----------|-----------|
| Local newer than remote | Push local changes |
| Remote newer than local | Pull remote changes |
| Both changed | Prompt user, default to remote |
| New in both (name conflict) | Merge fields, prefer remote for status |