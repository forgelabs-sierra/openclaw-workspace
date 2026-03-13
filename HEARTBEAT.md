# HEARTBEAT.md

## General check

Check: Any blockers, opportunities, or progress updates needed?

## Email check (every heartbeat ~hourly)

Check sierra@forgelabs.nz for unread emails from cameron@forgelabs.nz or matt@forgelabs.nz.

```bash
flgog gmail search 'from:(cameron@forgelabs.nz OR matt@forgelabs.nz) is:unread newer_than:2h' --max 10 --json --results-only
```

If emails found, for each email:

1. Read and understand the request
2. Classify the action needed:

   **Tier 1 — Do immediately, report results:**
   Read-only / information gathering (check status, list resources, search, query).
   Examples: "what version is X", "when did Y deploy", "check DNS for Z".

   **Tier 2 — Do it, report what you changed:**
   State-changing but low-risk and easily reversible (create a draft, add a tag).
   Test: *can this be undone trivially?*

   **Tier 3 — Make a plan, seek approval via Telegram:**
   State-changing and hard to reverse or high-impact (delete, deploy, send email, modify production config).
   Test: *would the sender want to know before this happens?*
   Notify the **original sender** for approval — Cameron approves Cameron's requests, Matt approves Matt's.

3. Take action per the tier, or present your plan and wait for approval
4. Summarise what you found/did and notify via Telegram
5. Mark email as read: `flgog gmail batch modify <messageId> --remove UNREAD`

If no results → continue to next check.

## Workspace backup (every heartbeat)

Backup key workspace files to GitHub. Run this script:

```bash
cd /workspace/openclaw-workspace
git config user.name "Sierra"
git config user.email "sierra@forgelabs.nz"
cp /workspace/MEMORY.md .
cp /workspace/TOOLS.md .
cp /workspace/HEARTBEAT.md .
mkdir -p skills memory
for skill_dir in /workspace/skills/*/; do
  skill_name=$(basename "$skill_dir")
  mkdir -p "skills/$skill_name"
  cp -r "$skill_dir"* "skills/$skill_name/" 2>/dev/null || true
done
cp /workspace/memory/*.md memory/ 2>/dev/null || true
# Redact secrets
if [ -f skills/webdev/SKILL.md ]; then
  sed -i 's|[0-9]\{12\}-[a-z0-9]\{32\}\.apps\.googleusercontent\.com|<redacted>|g' skills/webdev/SKILL.md
  sed -i 's|GOCSPX-[A-Za-z0-9_-]\{28\}|<redacted>|g' skills/webdev/SKILL.md
fi
# Commit if changed
if git diff --quiet && git diff --staged --quiet; then
  echo "No changes"
else
  TIMESTAMP=$(TZ="Pacific/Auckland" date "+%Y-%m-%d %H:%M NZT")
  git add -A && git commit -m "Workspace backup $TIMESTAMP" && flgh push
fi
```

If backup fails (e.g. clone missing), re-clone: `flgh clone forgelabs-sierra/openclaw-workspace` then `cd /workspace/openclaw-workspace && git remote set-url origin https://github.com/forgelabs-sierra/openclaw-workspace.git`

If no issues → HEARTBEAT_OK.
