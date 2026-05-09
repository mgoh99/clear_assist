# ClearSpace Agent Framework Template

The standard starting template for any ClearSpace agent. Defines the file layout, instruction conventions, and tool integrations needed to get an agent running — same template across **Nanobot**, **Hermes**, and **Claude Code**.

This repo ships with a **chief-of-staff** example agent (Outlook, Teams, Calendar, Planner via Microsoft 365). Fork it, replace the example contents, and you have a new agent on the same framework.

---

## Why a template

Every agent we run at ClearSpace needs the same skeleton:

- A canonical instructions file the harness reads on launch
- A tools/command reference the agent consults before hitting any service
- A `context/` folder holding the operator's profile, goals, and key relationships
- Slash commands or recipes the operator can invoke
- A predictable place for outputs

This repo is that skeleton. Every new ClearSpace agent starts here.

---

## Supported harnesses

| Harness | Reads | Notes |
|---|---|---|
| Claude Code | `CLAUDE.md` (imports `AGENTS.md`) | Slash commands in `.claude/commands/` |
| Nanobot | `AGENTS.md` | |
| Hermes | `AGENTS.md` | |
| Codex / Aider / others | `AGENTS.md` | Standard convention |

`AGENTS.md` is the single source of truth. `CLAUDE.md` is a thin Claude Code-specific shim that imports it — keeps instructions in one place across every harness.

---

## File layout

```
clear_assist/
├── AGENTS.md             # Canonical agent instructions (all harnesses)
├── CLAUDE.md             # Claude Code entry point — imports AGENTS.md
├── tools.md              # Command reference (syntax, parsing rules, IDs)
├── README.md             # This file
├── context/
│   ├── about-me.md       # Operator background, voice, preferences
│   ├── goals.md          # Annual goals — agent prioritizes against these
│   ├── people.md         # Key relationships
│   └── scheduling.md     # Calendar rules and constraints
├── .claude/
│   ├── commands/         # Slash commands (Claude Code)
│   └── settings.local.json
└── output/               # Agent-generated artifacts (gitignored)
```

---

## Prerequisites

At minimum, install the harness you plan to use. For the included chief-of-staff example, also install the Microsoft 365 CLI.

```bash
# Claude Code
npm install -g @anthropic-ai/claude-code

# Microsoft 365 CLI (for the example chief-of-staff agent)
npm install -g @pnp/cli-microsoft365
m365 login
m365 status
```

Verify Graph access:

```bash
m365 request --url "https://graph.microsoft.com/v1.0/me" --output json
```

---

## Setup

### 1. Fork and rename

Clone the template, rename the directory to your agent's purpose.

### 2. Fill in `context/`

| File | What to fill in |
|---|---|
| `context/about-me.md` | Name, role, company, timezone, voice, internal domains |
| `context/goals.md` | Top-level goals the agent should optimize against |
| `context/people.md` | Key contacts with emails and context |
| `context/scheduling.md` | Booking rules and hard constraints |

### 3. Personalize `AGENTS.md`

Replace `YOUR_NAME` and `YOUR_TIMEZONE`. Adjust the **Hard Rules** for your operator. Add or remove rows in the **Connected Services** table.

### 4. Cache stable IDs in `tools.md`

The bottom of `tools.md` has an "IDs Reference" block. Run the lookup commands once and paste the IDs in (Archive folder, Planner plan/bucket IDs, frequent contacts' AAD IDs). Slash commands then run with no extra lookups.

### 5. Pick your harness

```bash
# Claude Code
cd clear_assist && claude

# Nanobot / Hermes — point the harness at this directory; AGENTS.md loads automatically
```

---

## Built-in slash commands (Claude Code)

| Command | What it does |
|---|---|
| `/triage_inbox` | One-pass Outlook triage — unsubscribe, archive, draft replies |
| `/contact_find <Name>` | Resolve email + AAD ID + Teams chat ID for a person |
| `/planner_add <Name>: <task>` | Create a Planner task assigned to a teammate |

Or speak naturally: `triage my inbox`, `what's on my calendar today`, `draft a reply to NAME about TOPIC`, `schedule a meeting with NAME next week`.

---

## Adapting the template for a new agent

The framework is the file layout and the rules in `AGENTS.md`. To repurpose:

1. **Swap context** — rewrite `context/*.md` for the new operator and goals.
2. **Swap tools** — replace the M365 sections in `tools.md` with the services your agent needs (Salesforce, Slack, Notion, Linear, etc.).
3. **Swap commands** — edit or replace files in `.claude/commands/`. Each `.md` filename becomes the slash command name.
4. **Update `AGENTS.md` Connected Services table** — match the tool reference.

Hard Rules and writing style stay unless you have a specific reason to change them.

---

## Adding services

To add Google Workspace, Todoist, Salesforce, etc.:

1. Add a row to the **Connected Services** table in `AGENTS.md`.
2. Add a section to `tools.md` with command syntax + parsing rules + gotchas.
3. Update or add slash commands in `.claude/commands/` to use the new service.

Always prefer a CLI over an MCP when both exist (per `AGENTS.md` § Connected Services).

---

## Tips

- **Be specific in `context/about-me.md`** — the agent mirrors that voice in every draft.
- **Update `context/people.md` as the agent finds new contacts.** `/contact_find` will backfill automatically.
- **Cache IDs in `tools.md` once.** Slash commands stay fast.
- **Edit slash commands freely.** They're plain Markdown — adapt them to your operating cadence.
- **Single source of truth.** Edit `AGENTS.md`, not `CLAUDE.md` — the latter just imports the former.
