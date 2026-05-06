Triage the Outlook inbox in one pass — unsubscribe from marketing, draft replies for action items, archive everything that's already settled.

## Step 1 — Fetch the inbox

```bash
m365 request --url "https://graph.microsoft.com/v1.0/me/mailFolders/inbox/messages?\$top=50&\$orderby=receivedDateTime desc&\$select=id,subject,from,receivedDateTime,isRead,bodyPreview,body,internetMessageHeaders" --output json 2>/dev/null > /tmp/inbox.json
```

Deduplicate by id. Process unread + today's messages first.

## Step 2 — Classify

| Category | Criteria | Action |
|---|---|---|
| **MARKETING** | Newsletters, promotions, no-reply senders, `List-Unsubscribe` header, `Precedence: bulk` | Unsubscribe + archive |
| **SHIPPING** | "Shipped", "order confirmed", "out for delivery" | Archive only — *but keep* if it has a pickup/locker code |
| **TRANSACTIONAL** | Receipts, 2FA codes, booking confirmations, order confirmations | Archive only |
| **ACTION** | Requires reply, decision, or follow-up | Draft reply (do not send) |
| **FYI** | Worth a glance, no reply needed | Archive |
| **SKIP** | Already actioned, recurring auto-mail | Leave |

**Never auto-action** (leave alone): schools, daycares, government, financial institutions, healthcare, legal, kids' activities, anything with a person replying directly. When unsure → skip.

## Step 3 — Marketing: unsubscribe + archive

For each MARKETING email:

1. Look for `List-Unsubscribe` header or "unsubscribe" / "opt out" link in body. If found, surface it for the user to click (no built-in browser available — flag the URL in the summary).
2. Archive via Graph move endpoint (see `tools.md` § Mail).

## Step 4 — Action: draft replies

For each ACTION email, draft a reply matching the writing style in `context/about-me.md`. Show the draft for approval — **never send** without explicit "yes/send/approved" (Hard Rule #1).

Once approved and sent, archive the original in the same turn (Hard Rule #8).

## Step 5 — Summary

Output a single block grouped by category:

```
Triage — <date>
  [N] MARKETING archived ([M] unsub links surfaced)
  [N] SHIPPING / TRANSACTIONAL archived
  [N] ACTION drafts ready for review:
    1. <Sender> — <subject>
    2. ...
  [N] FYI archived
  [N] SKIP left in inbox
```
