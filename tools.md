# Tools — Command Reference

Command syntax for every connected service listed in AGENTS.md. Read this file **before** invoking any tool whose syntax you don't have in-context. The service inventory (what exists and when to use it) lives in AGENTS.md § Connected Services.

---

## Microsoft (m365)

```bash
# ── PYTHON PARSING RULES (avoid bash errors) ──────────────────────────────────
# 1. NEVER use python3 -c "..." — nested quotes cause syntax errors. Always use heredoc.
# 2. NEVER use `m365 outlook mail list` or `m365 outlook mail move` — they output CLI help, not JSON.
#    Always use Graph API for listing AND moving/archiving:
#      # List inbox:
#      m365 request --url "https://graph.microsoft.com/v1.0/me/mailFolders/inbox/messages?\$top=25&\$orderby=receivedDateTime desc&\$select=id,subject,from,receivedDateTime,isRead,bodyPreview" --output json 2>/dev/null > /tmp/msgs.json
#      # Archive a message (use Graph move endpoint, NOT `m365 outlook mail move`):
#      m365 request --method post --url "https://graph.microsoft.com/v1.0/me/messages/<messageId>/move" --body '{"destinationId":"<archiveFolderId>"}' --content-type "application/json" --output json 2>/dev/null
#      # Get archive folder ID first (cache it for the session):
#      m365 request --url "https://graph.microsoft.com/v1.0/me/mailFolders?\$filter=displayName eq 'Archive'" --output json 2>/dev/null
# 3. Array outputs (teams chat list, etc): pipe to file first — piping arrays to heredoc
#    fails with "null is not defined". Pattern:
#      m365 teams chat list --output json 2>/dev/null > /tmp/out.json && python3 << 'EOF'
#      import json
#      data = json.load(open('/tmp/out.json'))
#      EOF
# 4. Single-object outputs (m365 request): pipe to file, then parse — use raw.find('{') to skip header:
#      m365 request --url "..." --output json 2>/dev/null > /tmp/out.json && python3 << 'EOF'
#      import json
#      raw = open('/tmp/out.json').read()
#      d = json.loads(raw[raw.find('{'):])
#      EOF
# 5. POST/PUT with complex bodies: use subprocess + json.dumps() — avoids all shell quoting:
#      python3 << 'EOF'
#      import subprocess, json
#      body = json.dumps({"key": "value"})
#      r = subprocess.run(['m365','request','--method','post','--url','<url>','--body',body,'--content-type','application/json','--output','json'], capture_output=True, text=True)
#      print('OK' if r.returncode == 0 else r.stderr[:200])
#      EOF
```

### Mail

```bash
# Send email
m365 outlook mail send --subject "Subject" --to user@company.com --bodyContents "Body text"

# Search across mailbox
m365 search --scopes 'message' --queryText 'from:someone@company.com subject:invoice'

# Read inbox (raw Graph for reliable JSON)
m365 request --url "https://graph.microsoft.com/v1.0/me/mailFolders/inbox/messages?\$top=25&\$orderby=receivedDateTime desc&\$select=id,subject,from,receivedDateTime,isRead,bodyPreview" --output json 2>/dev/null > /tmp/msgs.json

# Read a single message body
m365 request --url "https://graph.microsoft.com/v1.0/me/messages/<messageId>?\$select=subject,from,toRecipients,body,receivedDateTime" --output json 2>/dev/null

# Reply to a message
m365 request --method post --url "https://graph.microsoft.com/v1.0/me/messages/<messageId>/reply" --body '{"message":{"body":{"contentType":"Text","content":"Reply text"}}}' --content-type "application/json"

# Archive (move to Archive folder)
m365 request --method post --url "https://graph.microsoft.com/v1.0/me/messages/<messageId>/move" --body '{"destinationId":"<archiveFolderId>"}' --content-type "application/json"
```

### Calendar

```bash
# Lookup calendar events for a date window
m365 request --url "https://graph.microsoft.com/v1.0/me/calendarView?startDateTime=<DATE>T00:00:00Z&endDateTime=<DATE>T23:59:59Z&\$orderby=start/dateTime&\$top=200&\$select=subject,start,end,isAllDay,isCancelled,showAs,location,organizer,isOrganizer,attendees" --output json
# ⚠️ Graph returns datetimes with 7-digit fractional seconds ("2026-04-23T14:30:00.0000000") — Python <3.11
# fromisoformat() throws "Invalid isoformat string". Strip the fractional part before parsing:
#   s = e['start']['dateTime'].replace('Z','').split('.')[0]
#   start_utc = datetime.fromisoformat(s).replace(tzinfo=timezone.utc)
# Graph returns UTC — convert to YOUR_TIMEZONE before doing anything with it.

# Create calendar event
m365 request --method post --url "https://graph.microsoft.com/v1.0/me/events" --body '{"subject":"Meeting","start":{"dateTime":"<DATE>T10:00:00","timeZone":"Eastern Standard Time"},"end":{"dateTime":"<DATE>T11:00:00","timeZone":"Eastern Standard Time"},"attendees":[{"emailAddress":{"address":"user@company.com","name":"Name"},"type":"required"}]}' --content-type "application/json"

# Update event
m365 request --method patch --url "https://graph.microsoft.com/v1.0/me/events/<eventId>" --body '{"subject":"New title"}' --content-type "application/json"

# Cancel event
m365 request --method post --url "https://graph.microsoft.com/v1.0/me/events/<eventId>/cancel" --body '{"comment":"Cancelling"}' --content-type "application/json"
```

### Teams

```bash
# Find a user (AAD ID)
m365 request --url "https://graph.microsoft.com/v1.0/users?\$filter=startswith(displayName,'NAME')&\$select=displayName,mail,id" --output json 2>/dev/null

# Find an existing 1:1 chat (NEVER use --userEmails — always use chatId)
m365 teams chat list --type oneOnOne --output json 2>/dev/null > /tmp/chats.json
# (then match the chat by member AAD ID in /tmp/chats.json)

# Send a Teams DM
m365 teams chat message send --chatId "<chatId>" --message "Hello"

# Send a message to a Teams channel
m365 teams chat message send --teamId "<teamId>" --channelId "<channelId>" --message "Hello"
```

### Planner

```bash
# List plans for a group
m365 request --url "https://graph.microsoft.com/v1.0/groups/<groupId>/planner/plans" --output json

# List buckets for a plan
m365 request --url "https://graph.microsoft.com/v1.0/planner/plans/<planId>/buckets" --output json

# Create a task (assignment requires AAD ID — see Teams section)
python3 << 'EOF'
import subprocess, json
body = json.dumps({
    "planId": "<planId>",
    "bucketId": "<bucketId>",
    "title": "<task title>",
    "assignments": {
        "<aad-id>": {"@odata.type": "#microsoft.graph.plannerAssignment", "orderHint": " !"}
    }
})
r = subprocess.run(['m365','request','--method','post','--url','https://graph.microsoft.com/v1.0/planner/tasks','--body',body,'--content-type','application/json','--output','json'], capture_output=True, text=True)
print('OK' if r.returncode == 0 else r.stderr[:200])
EOF
```

---

## Playwright (`playwright`)

Headless browser automation. Reach for it when a page needs JavaScript rendering, authentication, or interaction (clicks, form fills, waits) that a plain HTTP fetch can't handle. For one-off captures, the CLI subcommands below are enough; for anything multi-step or repeatable, write a Node/Python script against the Playwright library instead of chaining CLI calls.

Invoke via `npx playwright <command>` (no global install required). First run on a fresh machine triggers a browser download — run `npx playwright install` once if commands fail with "browser not installed."

### Common commands

```bash
# Screenshot a page (full page = whole scroll height, not just viewport)
npx playwright screenshot --full-page https://example.com /tmp/page.png

# Save page as PDF (Chromium only)
npx playwright pdf https://example.com /tmp/page.pdf

# Open a visible browser pointed at a URL — useful for inspecting state interactively
npx playwright open https://example.com

# Codegen — record clicks/fills in a visible browser and emit a Playwright script
npx playwright codegen https://example.com

# Install / update browser binaries (one-time per machine; re-run after Playwright upgrades)
npx playwright install
```

### Notes & gotchas

- **Prefer CLI > MCP > Playwright.** Playwright is the heaviest option — only use it when lighter tools can't see the content (JS-rendered SPA, login-walled page, etc.).
- **Default device:** the CLI loads Chromium with a desktop viewport. Pass `--device "iPhone 14"` (or any [Playwright device name](https://playwright.dev/docs/emulation)) to emulate mobile.
- **Auth state:** the `screenshot`/`pdf`/`open` subcommands start from a clean profile every time — no cookies, no logged-in session. For authenticated capture, write a script that loads a saved `storageState.json`.
- **Save outputs under `output/`**, per the AGENTS.md output policy (e.g. `output/screenshots/<name>.png`), not `/tmp/`, unless the file is genuinely throwaway.
- **Headless by default.** Add `--browser firefox` or `--browser webkit` to test cross-browser; omit for Chromium.

---

## IDs Reference (fill in once, reuse forever)

Cache stable IDs here so commands above don't rely on lookups every time. Update when teams/plans change.

```
Archive folder ID:        <fill in once via the mailFolders query>
Planner — Plan IDs:       <board name>: <plan id>
Planner — Bucket IDs:     <plan>/<bucket>: <bucket id>
Teams — Channel IDs:      <team>/<channel>: <channel id>
Frequent contacts AAD:    NAME: <aad id>
```
