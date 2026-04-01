# Claude Chief of Staff

A Claude Code workspace that acts as your AI chief of staff — handling email triage, calendar management, task tracking, drafting, research, and more through connected services.

Claude operates here as a productivity partner: reads your context files, uses connected CLIs, and executes tasks across your work and personal services.

---

## What it does

- **Inbox triage** — classify, unsubscribe, archive, and draft replies across work + personal email
- **Calendar management** — check availability, schedule meetings, find conflicts
- **Task management** — read and update your Todoist inbox
- **Drafting** — emails, messages, intros, and documents in your voice
- **Research** — web browsing, scraping, and interaction via Playwright
- **People lookup** — find contacts by searching your email before asking you

---

## Prerequisites

You'll need Claude Code installed:
```bash
npm install -g @anthropic/claude-code
```

---

## Services & Setup

### 1. Microsoft 365 (work email, calendar, Teams)

Used for Outlook email, calendar, and Teams messages.

**Install:**
```bash
npm install -g @pnp/cli-microsoft365
m365 login
```

Follow the device code login flow. Once authenticated:
```bash
m365 status  # should show your account
```

**Verify:**
```bash
m365 request --url "https://graph.microsoft.com/v1.0/me" --output json
```

---

### 2. Google Workspace CLI (personal Gmail + Google Calendar)

Used for personal email and calendar.

**Install:**
```bash
npm install -g @googleapis/gws-cli
# or follow: https://github.com/googleworkspace/gws-cli
gws login
```

Once authenticated, verify:
```bash
gws gmail users getProfile --params '{"userId":"me"}'
```

---

### 3. Playwright (web browsing + browser interaction)

Used for navigating pages, clicking, filling forms, extracting content, and taking screenshots. No API key required — runs locally.

**Install:**
```bash
npm install -g @playwright/mcp
npx playwright install chromium  # install the browser
```

**Add to Claude Code settings** (`~/.claude/settings.json`):
```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp"]
    }
  }
}
```

**Verify** — restart Claude Code and check that browser tools appear (navigate, click, screenshot, etc.).

> **Why Playwright over Firecrawl:** No API key, no rate limits, fully local, handles JavaScript-heavy pages, and can interact with any web UI including forms and logins.

---

## Configuration

### 1. Fill in your context files

Copy and fill in each file in `context/`:

| File | What to fill in |
|------|----------------|
| `context/about-me.md` | Your name, role, company, timezone, communication style |
| `context/goals.md` | Your annual goals — Claude uses these to prioritize |
| `context/people.md` | Key people in your life with emails and context |
| `context/scheduling.md` | Your scheduling preferences and constraints |

### 2. Update CLAUDE.md

Edit `CLAUDE.md` to:
- Replace `YOUR_NAME` with your name
- Replace `YOUR_EMAIL_WORK` and `YOUR_EMAIL_PERSONAL` with your emails
- Update file paths if your `context/` folder lives somewhere else
- Remove or add services you don't use

### 3. (Optional) Set up Todoist

If you use Todoist, add the MCP server to your Claude Code settings:

```json
// ~/.claude/settings.json
{
  "mcpServers": {
    "todoist": {
      "command": "npx",
      "args": ["-y", "todoist-mcp-server"],
      "env": {
        "TODOIST_API_TOKEN": "your_token_here"
      }
    }
  }
}
```

Get your API token at [todoist.com/app/settings/integrations](https://todoist.com/app/settings/integrations/developer).

---

## Usage

Open Claude Code in this directory:
```bash
cd clear_assist
claude
```

Example prompts:
- `triage my inbox`
- `what's on my calendar today`
- `draft a reply to [person] about [topic]`
- `schedule a meeting with [person] next week`
- `add a task: [task description]`

---

## File structure

```
clear_assist/
├── CLAUDE.md              # Main instructions for Claude
├── README.md              # This file
├── context/
│   ├── about-me.md        # Your background and preferences
│   ├── goals.md           # Your annual goals
│   ├── people.md          # Key contacts
│   └── scheduling.md      # Scheduling rules
└── scripts/               # Optional helper scripts
```

---

## Tips

- **Be specific in CLAUDE.md** — the more context Claude has about your preferences and style, the less you need to explain each session.
- **Keep `people.md` updated** — Claude will search your email to find unknown contacts and add them here automatically.
- **Use the `/triage_inbox` pattern** — create a `.claude/commands/triage_inbox.md` skill file to define a repeatable inbox triage flow. See [Claude Code skills docs](https://docs.anthropic.com/claude-code).
