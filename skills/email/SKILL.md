---
name: email
description: Draft, review, and send emails from sierra@forgelabs.nz via flgog. Always CC cameron@forgelabs.nz. Always get explicit approval before sending. Use when composing or sending any email on behalf of Forge Labs.
metadata: {"clawdbot":{"emoji":"📧","requires":{"bins":["flgog"]}}}
---

# email — Draft, Confirm & Send

Handles the full email lifecycle from sierra@forgelabs.nz: compose → show Cameron → get approval → send.

## Rules (non-negotiable)

- **Always send from** sierra@forgelabs.nz (default flgog account)
- **Always CC** cameron@forgelabs.nz — no exceptions unless Cameron explicitly says not to
- **Never send without explicit approval** — must hear "send it" or equivalent confirmation
- **Never guess** on recipient, subject, or tone — confirm if unclear
- **Keep draft ID** in working memory until sent or cancelled

## Workflow

### Step 1 — Compose

Gather the required details:
- `--to` recipient(s)
- `--subject`
- body content

Write the body to a temp file in /workspace to avoid shell quoting issues:

```bash
# Write body to temp file
cat > /workspace/email-draft.txt << 'EOF'
Body content here...
EOF
```

### Step 2 — Create Draft

Always include CC to cameron@forgelabs.nz:

```bash
BODY=$(cat /workspace/email-draft.txt)
flgog gmail draft create \
  --to recipient@example.com \
  --cc cameron@forgelabs.nz \
  --subject "Subject here" \
  --body "$BODY"
```

Note the `draft_id` from the output (format: `r-XXXXXXXXXXXXXXXXX`).

Clean up the temp file immediately after:
```bash
rm /workspace/email-draft.txt
```

### Step 3 — Show Cameron

Do NOT retrieve the draft via `flgog gmail draft get` — just display the draft content you already have.

Present the email inline in the chat, chunked for Telegram readability. Telegram truncates long messages, so break the draft into logical pieces — typically:
- Chunk 1: To / CC / Subject
- Chunk 2: Body (split further if long)

Send each chunk as a separate message. Do not label the chunks (no "Chunk 1/3" etc — just send them in sequence).

Then ask:
> "Happy with this? Any changes, or shall I send it?"

### Step 4 — Handle Response

**If approved** ("send it", "go ahead", "yes", etc.) → go to Step 5.

**If changes requested:**
1. Delete the old draft: `flgog gmail draft delete <draft_id>`
2. Rewrite with changes
3. Create new draft (Step 2) — note new draft_id
4. Show updated draft (Step 3)
5. Wait for approval again

**If cancelled** ("forget it", "cancel", "don't send"):
1. Delete the draft: `flgog gmail draft delete <draft_id>`
2. Confirm cancelled

### Step 5 — Send

```bash
flgog gmail draft send <draft_id>
```

Confirm to Cameron: "Sent! ✉️ [recipient] — subject line"

## Draft Commands Reference

```bash
# Create draft
flgog gmail draft create --to addr --cc cameron@forgelabs.nz --subject "Subject" --body "Body"

# View draft
flgog gmail draft get <draft_id>

# List all drafts
flgog gmail draft list

# Send draft
flgog gmail draft send <draft_id>

# Delete draft
flgog gmail draft delete <draft_id>
```

## Attachments

```bash
flgog gmail draft create \
  --to recipient@example.com \
  --cc cameron@forgelabs.nz \
  --subject "Subject" \
  --body "$BODY" \
  --attach "/path/to/file.pdf"
```

**Important — host path required:** `flgog` runs on the host (macOS), not inside the sandbox. The sandbox `/workspace` is a virtiofs mount not accessible from the host at that path. Use the host path:

```
Sandbox path:  /workspace/myfile.zip
Host path:     /Users/forgelabs/forgelabs/projects/fl/ai/openclaw-podman/openclaw-data/workspace/myfile.zip
```

To create a file in the sandbox and attach it via flgog, write it to `/workspace/` then reference it at the host path above.

## Multiple Recipients

```bash
# Comma-separated for multiple To/CC
flgog gmail draft create \
  --to "matt@forgelabs.nz,other@example.com" \
  --cc cameron@forgelabs.nz \
  --subject "Subject" \
  --body "$BODY"
```

## Reply to an existing thread

```bash
# Use --reply-to-message-id to thread correctly
flgog gmail draft create \
  --to recipient@example.com \
  --cc cameron@forgelabs.nz \
  --subject "Re: Original subject" \
  --reply-to-message-id <gmail_message_id> \
  --body "$BODY"
```

## Tone & Style

- Write as Sierra unless Cameron specifies otherwise
- Professional but warm — Forge Labs voice
- Sign off as "Sierra" or "The Forge Labs Team" unless told otherwise
- Keep emails concise — respect the reader's time

## Checklist Before Sending

- [ ] Recipient correct?
- [ ] CC cameron@forgelabs.nz included?
- [ ] Subject clear and descriptive?
- [ ] Tone appropriate?
- [ ] Cameron has approved?
