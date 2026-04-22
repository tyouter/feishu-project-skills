---
name: update
description: 'Version-aware incremental update for installed skills. Checks source repo for new versions, shows changelog, backs up and updates SKILL.md files. Trigger: /update'
trigger: /update
---

# /update

Version-aware incremental update for installed skills. Compares the source repo VERSION against the installed version, shows changes, and updates SKILL.md files while preserving user data.

## Commands

```
/update              # Check and perform update
/update --check      # Check only (no changes)
/update --changelog  # Show CHANGELOG
/update --force      # Force update (skip confirmation)
/update --rollback <version>  # Rollback to a specific version
```

## Shared Variables

- `SOURCE_REPO` — The feishu-project-skills repo path (read from `.claude/skills-source.json → source_repo`)
- `VAULT_ROOT` — The target vault root (current working directory)
- `INSTALLED_VERSION` — Currently installed version (from skills-source.json)
- `LATEST_VERSION` — Latest version in source repo (from SOURCE_REPO/VERSION)

## Helper: Read Source Repo Path

The source repo path is stored in `VAULT_ROOT/.claude/skills-source.json`. If this file does not exist, tell the user:

> skills-source.json not found. Please run `/setup` first to initialize this vault.

If the file exists but `source_repo` path is invalid (directory does not exist), ask the user:

> The source repo path `<source_repo>` no longer exists. Please provide the updated path to feishu-project-skills:

Update `source_repo` in skills-source.json after user provides a valid path.

## Workflow

### Phase 1: Version Detection

1. Read `VAULT_ROOT/.claude/skills-source.json` to get `source_repo` and `installed_version`
2. If file does not exist → "skills-source.json not found, run `/setup` first" and stop
3. Read `SOURCE_REPO/VERSION` to get `LATEST_VERSION`
4. Compare versions:
   - `INSTALLED_VERSION == LATEST_VERSION` → "Already at latest version X.X.X" and stop
   - `INSTALLED_VERSION != LATEST_VERSION` → proceed to Phase 2

If `--check` flag was used, stop here after showing the version comparison result.

### Phase 2: Change Display

1. Read `SOURCE_REPO/CHANGELOG.md`
2. Extract entries between `INSTALLED_VERSION` and `LATEST_VERSION`
3. Display the changelog entries
4. List files that will be updated (SKILL.md files in `.claude/skills/`)
5. Check for new template files:
   - Read `installed_templates` from skills-source.json
   - Scan `SOURCE_REPO/templates/` for any `.example.*` files
   - Compare against what was recorded in `installed_templates` (if exists) in skills-source.json
   - List any new templates detected
6. Ask user to confirm:
   - `Confirm update` → Phase 3 (full update including new templates)
   - `Skills only` → Phase 3 (skip Phase 3c new template detection)
   - `Cancel` → stop

If `--force` flag is used, skip confirmation and proceed directly to Phase 3.

### Phase 3: Execute Update

#### Phase 3a: Backup Current Skills

Create a backup directory:

```
VAULT_ROOT/.claude/backups/<installed_version>_<timestamp>/
```

Where timestamp is `YYYYMMDDHHmmss` format (compact, no colons).

Copy all SKILL.md files from `VAULT_ROOT/.claude/skills/` to the backup directory, preserving subdirectory structure:

```bash
BACKUP_DIR="${VAULT_ROOT}/.claude/backups/${INSTALLED_VERSION}_$(date +%Y%m%d%H%M%S)"
mkdir -p "${BACKUP_DIR}"

# Copy each skill's SKILL.md to backup
for skill_dir in "${VAULT_ROOT}/.claude/skills"/*/; do
  skill_name=$(basename "${skill_dir}")
  if [ -f "${skill_dir}/SKILL.md" ]; then
    mkdir -p "${BACKUP_DIR}/${skill_name}"
    cp "${skill_dir}/SKILL.md" "${BACKUP_DIR}/${skill_name}/SKILL.md"
  fi
done
```

Record the backup version in `skills-source.json → backup_versions` array.

#### Phase 3b: Update SKILL.md Files

Copy all skill directories from source repo to target vault:

```bash
SKILLS_SRC="${SOURCE_REPO}/.claude/skills"

# Copy all skills (including update/ which may be new)
for skill_dir in "${SKILLS_SRC}"/*/; do
  skill_name=$(basename "${skill_dir}")
  mkdir -p "${VAULT_ROOT}/.claude/skills/${skill_name}"
  cp "${skill_dir}/SKILL.md" "${VAULT_ROOT}/.claude/skills/${skill_name}/SKILL.md"
done
```

This overwrites SKILL.md files but does NOT touch any other files in the skill directories.

#### Phase 3c: Detect New Templates

Only if user chose "Confirm update" (not "Skills only"):

1. Scan `SOURCE_REPO/templates/` for all `.example.*` files
2. For each template file:
   - Strip `.example` from the filename to get the target name (e.g., `sync-mapping.example.yaml` → `sync-mapping.yaml`)
   - Check if target file already exists in `VAULT_ROOT/.claude/`
   - If target exists → skip (never overwrite user files)
   - If target does NOT exist → ask user:
     > New template detected: `<filename>`. Generate it in vault? (Will not overwrite existing files)
   - If user agrees → create from template (copy the `.example.*` file to `VAULT_ROOT/.claude/<stripped_name>`)
   - Record in `installed_templates` array in skills-source.json

#### Phase 3d: Update skills-source.json

Update the file with:

```json
{
  "source_repo": "<keep existing>",
  "installed_version": "<LATEST_VERSION>",
  "install_level": "<keep existing>",
  "last_setup": "<keep existing>",
  "last_update": "<current ISO timestamp>",
  "installed_skills": ["<list all skill dirs now present>"],
  "installed_templates": ["<list all templates that were installed>"],
  "backup_versions": ["<append current version to existing array>"]
}
```

Read the existing file, update only the relevant fields, write back.

#### Phase 3e: Update .gitignore (if needed)

1. Read `SOURCE_REPO/init/vault-skeleton/.gitignore`
2. Read `VAULT_ROOT/.gitignore` (if exists)
3. For each line in source .gitignore:
   - If the line is not in target .gitignore → append it
   - Do NOT remove any lines from target .gitignore
4. If changes were made, inform the user what was added

### Phase 4: Report

Display a summary:

```
## Update Complete

Version: <old> → <new>
Updated skills: <N> skills
New templates: <list or "none">
Backup location: .claude/backups/<version>_<timestamp>/

Rollback: To restore the previous version, run /update --rollback <old_version>
```

## Rollback Workflow (`/update --rollback <version>`)

1. Verify backup exists at `VAULT_ROOT/.claude/backups/<version>_*/`
2. If not found → "No backup found for version <version>. Available: <list>" and stop
3. If multiple backups match (different timestamps), use the latest one
4. Show what will be restored (list of SKILL.md files in the backup)
5. Ask for confirmation
6. Copy all SKILL.md files from backup to `VAULT_ROOT/.claude/skills/`
7. Update `skills-source.json → installed_version` to the rollback version
8. Report:
   ```
   ## Rollback Complete

   Restored to version: <version>
   From backup: .claude/backups/<version>_<timestamp>/
   ```

## Changelog Workflow (`/update --changelog`)

1. Read `SOURCE_REPO/CHANGELOG.md` (resolve source from skills-source.json)
2. Display the full changelog content
3. If source_repo is unavailable, try reading from `VAULT_ROOT/.claude/skills/update/` — this SKILL.md itself may contain version info in comments

## Safety Rules

- NEVER overwrite user data files: `.env`, `team-registry.json`, `sync-state.yaml`, `sync-mapping.yaml`, `feishu-local.yaml`, `managed-projects.yaml`
- NEVER delete backup directories (only append new ones)
- Always backup before updating
- If any step fails (e.g., source repo not found), stop and inform the user — do not proceed with partial updates
