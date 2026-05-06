Find contact info (email, AAD ID, Teams chat ID) for a person. Argument: the person's name (e.g. `/contact_find Chris Ho`).

## Instructions

Fire all searches in parallel via separate Bash calls. No narration — just execute and report.

1. **M365 directory:**
   ```bash
   m365 request --url "https://graph.microsoft.com/v1.0/users?\$filter=startswith(displayName,'$ARGUMENTS')&\$select=displayName,mail,id" --output json 2>/dev/null
   ```

2. **Calendar attendees (last 14d + next 30d):**
   ```bash
   m365 request --url "https://graph.microsoft.com/v1.0/me/calendarView?startDateTime=<14d-ago>T00:00:00Z&endDateTime=<30d-ahead>T23:59:59Z&\$top=200&\$select=subject,attendees" --output json 2>/dev/null
   ```
   Scan attendee names/emails for a match.

3. **Outlook search:**
   ```bash
   m365 search --scopes 'message' --queryText '$ARGUMENTS' --output json 2>/dev/null
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
$ARGUMENTS
  Email: <email>
  AAD ID: <id>
  Teams Chat ID: <chatId>
  Source: <directory|calendar|outlook>
```

One line per field. "not found" if nothing across all searches. No extra commentary.

## Write-back

If a match is found and the person isn't yet in `context/people.md`, add them under the right section before reporting. Hard Rule #1 doesn't apply — file edits are local, not outbound.
