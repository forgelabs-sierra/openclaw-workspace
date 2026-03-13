---
name: workspace-backup
description: Backup Sierra's workspace files (MEMORY.md, TOOLS.md, HEARTBEAT.md, skills/, memory/) to GitHub repo forgelabs-sierra/openclaw-workspace. Run on each heartbeat cycle.
---

# workspace-backup — Automated Workspace Backup to GitHub

Commits and pushes key workspace files to `forgelabs-sierra/openclaw-workspace` on every heartbeat.
Provides a recovery point if workspace files are corrupted or zeroed.

## Backup repo

- **GitHub:** https://github.com/forgelabs-sierra/openclaw-workspace
- **Local clone:** /workspace/openclaw-workspace

## Files backed up

- `/workspace/MEMORY.md`
- `/workspace/TOOLS.md`
- `/workspace/HEARTBEAT.md`
- `/workspace/skills/` (all SKILL.md files and references)
- `/workspace/memory/` (all daily log files)

## Security: redact secrets before push

Some skill files may contain credentials (e.g. webdev/SKILL.md has GCP OAuth values).
Always redact before pushing to GitHub. The backup script handles this automatically.

Known secret patterns to redact in the backup copy:
- `skills/webdev/SKILL.md`: AUTH_GOOGLE_ID, AUTH_GOOGLE_SECRET values

## Backup procedure

Run this script on each heartbeat:

```bash
#!/bin/bash
set -e

BACKUP_DIR=/workspace/openclaw-workspace
WORKSPACE=/workspace
TIMESTAMP=$(TZ="Pacific/Auckland" date "+%Y-%m-%d %H:%M NZT")

# Ensure backup repo exists and is on main
cd "$BACKUP_DIR"
git config user.name "Sierra"
git config user.email "sierra@forgelabs.nz"

# Copy files
cp "$WORKSPACE/MEMORY.md" .
cp "$WORKSPACE/TOOLS.md" .
cp "$WORKSPACE/HEARTBEAT.md" .

mkdir -p skills memory

# Copy skills (all subdirs)
cp -r "$WORKSPACE"/skills/*/SKILL.md . 2>/dev/null || true
# Proper copy preserving structure:
for skill_dir in "$WORKSPACE"/skills/*/; do
  skill_name=$(basename "$skill_dir")
  mkdir -p "skills/$skill_name"
  cp -r "$skill_dir"* "skills/$skill_name/" 2>/dev/null || true
done

# Copy memory logs
cp "$WORKSPACE"/memory/*.md memory/ 2>/dev/null || true

# Redact known secrets from backup copies
# webdev skill: GCP OAuth credentials
if [ -f skills/webdev/SKILL.md ]; then
  sed -i 's|[0-9]\{12\}-[a-z0-9]\{32\}\.apps\.googleusercontent\.com|<redacted — see Vercel env vars>|g' skills/webdev/SKILL.md
  sed -i 's|GOCSPX-[A-Za-z0-9_-]\{28\}|<redacted — see Vercel env vars>|g' skills/webdev/SKILL.md
fi

# Check if anything changed
if git diff --quiet && git diff --staged --quiet; then
  echo "No changes to backup"
  exit 0
fi

# Commit and push
git add -A
git commit -m "Workspace backup $TIMESTAMP"
flgh push
echo "Backup complete: $TIMESTAMP"
```

## Heartbeat integration

This skill runs automatically on each heartbeat. The HEARTBEAT.md entry is:

```
## Workspace backup (every heartbeat)
Run the workspace backup script to commit and push any changes to GitHub.
```

## Setup notes

- Local clone at `/workspace/openclaw-workspace` must exist with correct remote
- Remote URL must be `https://github.com/forgelabs-sierra/openclaw-workspace.git` (no token in URL)
- If the clone is missing, re-clone with: `flgh clone forgelabs-sierra/openclaw-workspace` then fix remote URL

## Recovery

If workspace files are corrupted:
1. `flgh clone forgelabs-sierra/openclaw-workspace /tmp/restore`
2. `cp /tmp/restore/MEMORY.md /workspace/MEMORY.md`
3. `cp /tmp/restore/TOOLS.md /workspace/TOOLS.md`
4. etc.
