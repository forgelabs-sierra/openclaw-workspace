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

If no results → HEARTBEAT_OK.
