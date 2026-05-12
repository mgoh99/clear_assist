---
name: contact-find
description: Resolve a person's email, AAD ID, and Teams chat ID via Microsoft Graph. Use when the user asks to find contact info, look up an email, get someone's AAD/Teams ID, or says "find <Name>", "who is <Name>", "look up <Name>'s email", "get me <Name>'s Teams chat". Searches M365 directory, calendar attendees, and Outlook in parallel.
---

# Contact find

Resolve contact info (email, AAD ID, Teams chat ID) for a person by name across Microsoft Graph.

## Instructions

Fire all searches in parallel via separate Bash calls. No narration — just execute and report.

1. **M365 directory:**
   ```bash
   m365 request --url "https://graph.microsoft.com/v1.0/users?\$filter=startswith(displayName,'<Name>')&\$select=displayName,mail,id" --output json 2>/dev/null
   ```

2. **Calendar attendees (last 14d + next 30d):**
   ```bash
   m365 request --url "https://graph.microsoft.com/v1.0/me/calendarView?startDateTime=<14d-ago>T00:00:00Z&endDateTime=<30d-ahead>T23:59:59Z&\$top=200&\$select=subject,attendees" --output json 2>/dev/null
   ```
   Scan attendee names/emails for a match.

3. **Outlook search:**
   ```bash
   m365 search --scopes 'message' --queryText '<Name>' --output json 2>/dev/null
   ```
   Extract from/to emails matching the name.

4. **Teams chat ID** — only after step 1 returns an AAD ID:
   ```bash
   m365 teams chat list --type oneOnOne --output json 2>/dev/null
   ```
   Match by user ID.

Steps 1–3 run in parallel. Step 4 runs after.

## Output

```
<Name>
  Email: <email>
  AAD ID: <id>
  Teams Chat ID: <chatId>
  Source: <directory|calendar|outlook>
```

One line per field. "not found" if nothing across all searches. No extra commentary.

## Write-back

If a match is found and the person isn't yet in `context/people.md`, add them under the right section before reporting. Hard Rule #1 doesn't apply — file edits are local, not outbound.
