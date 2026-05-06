# Clear Assist

A Claude Code workspace that acts as your AI chief of staff — handling email triage, calendar management, drafting, and research through Microsoft 365.

This is a **template**. Clone it, fill in the placeholders in `context/` and `CLAUDE.md`, and you have a personal CoS workspace operating against your Outlook, calendar, Teams, and Planner.

---

## What it does

- **Inbox triage** — classify, unsubscribe, archive, and draft replies in Outlook
- **Calendar management** — check availability, schedule meetings
- **Drafting** — emails, Teams messages, intros, documents in your voice
- **Planner tasks** — assign work to teammates from a one-line command
- **People lookup** — find contacts by searching directory + calendar + email

All of this is driven by the slash commands in `.claude/commands/`. Add or edit them to fit your operating model.

---

## Prerequisites

Claude Code installed:
```bash
npm install -g @anthropic-ai/claude-code
```

---

## Setup

### 1. Microsoft 365 CLI

```bash
npm install -g @pnp/cli-microsoft365
m365 login
m365 status   # confirm your account
```

Verify Graph access:
```bash
m365 request --url "https://graph.microsoft.com/v1.0/me" --output json
```

### 2. Fill in your context

Edit each file in `context/`:

| File | What to fill in |
|---|---|
| `context/about-me.md` | Name, role, company, timezone, communication style, internal domains |
| `context/goals.md` | Annual goals — Claude prioritizes against these |
| `context/people.md` | Key people with emails and context |
| `context/scheduling.md` | Booking rules, hard constraints |

### 3. Personalize `CLAUDE.md`

Replace `YOUR_NAME` and `YOUR_TIMEZONE` throughout. Adjust Hard Rules to your context. Add or remove services in the Connected Services table.

### 4. Cache stable IDs in `tools.md`

The bottom of `tools.md` has an "IDs Reference" section. Run the lookup commands once and paste the IDs in (Archive folder, Planner plan/bucket IDs, frequent contacts' AAD IDs). After that, slash commands run with no extra lookups.

---

## Usage

```bash
cd clear_assist
claude
```

Built-in slash commands:

| Command | What it does |
|---|---|
| `/triage_inbox` | Pass through Outlook — unsub, archive, draft replies |
| `/contact_find <Name>` | Find email + AAD ID + Teams chat ID for a person |
| `/planner_add <Name>: <task>` | Add a Planner task assigned to a teammate |

Or just speak naturally:
- `triage my inbox`
- `what's on my calendar today`
- `draft a reply to NAME about TOPIC`
- `schedule a meeting with NAME next week`

---

## File structure

```
clear_assist/
├── CLAUDE.md              # Main instructions for Claude
├── tools.md               # Command reference (M365 syntax + parsing rules + IDs)
├── README.md              # This file
├── context/
│   ├── about-me.md        # Background and preferences
│   ├── goals.md           # Annual goals
│   ├── people.md          # Key contacts
│   └── scheduling.md      # Scheduling rules
├── .claude/commands/      # Slash commands (templates — edit freely)
└── output/                # Generated artifacts (decks, datasets, write-ups)
```

---

## Adding services

This template ships with M365 only. To add Google Workspace, Todoist, Obsidian, etc.:

1. Add the service row to the Connected Services table in `CLAUDE.md`.
2. Add a section to `tools.md` with command syntax + parsing rules.
3. Update or add slash commands in `.claude/commands/` to use the new service.

---

## Tips

- **Be specific in `context/about-me.md`** — Claude mirrors that voice in every draft.
- **Update `context/people.md`** as Claude finds new contacts. Use `/contact_find` to backfill.
- **Cache IDs in `tools.md`** once. Slash commands stay fast.
- **Edit slash commands freely.** They're plain Markdown files. Adapt them to your operating cadence.
