# ClearSpace Agent Framework Template

The standard starting template for any ClearSpace agent. Defines the file layout, instruction conventions, and tool integrations needed to get an agent running — same template across **Nanobot**, **Hermes**, and **Claude Code**.

This repo ships with a **chief-of-staff** example agent (Outlook, Teams, Calendar, Planner via Microsoft 365). Fork it, replace the example contents, and you have a new agent on the same framework.

---

## Why a template

Every agent we run at ClearSpace needs the same skeleton:

- A canonical instructions file the harness reads on launch
- A tools/command reference the agent consults before hitting any service
- A `USER.md` file holding the operator's profile, preferences, and key relationships
- A `SOUL.md` file defining the agent's voice and operating principles
- Reusable skills the agent can invoke automatically when relevant
- A predictable place for outputs

This repo is that skeleton. Every new ClearSpace agent starts here.

---

## Supported harnesses

| Harness | Reads | Notes |
|---|---|---|
| Claude Code | `CLAUDE.md` (imports `AGENTS.md`) | Skills in `skills/<name>/SKILL.md` |
| Nanobot | `AGENTS.md` | Skills in `skills/<name>/SKILL.md` |
| Hermes | `AGENTS.md` | Skills in `skills/<name>/SKILL.md` |
| Codex / Aider / others | `AGENTS.md` | Standard convention |

`AGENTS.md` is the single source of truth. `CLAUDE.md` is a thin Claude Code-specific shim that imports it — keeps instructions in one place across every harness.

---

## File layout

```
clear_assist/
├── AGENTS.md             # Canonical agent instructions (all harnesses)
├── CLAUDE.md             # Claude Code entry point — imports AGENTS.md
├── USER.md               # Operator profile: identity, preferences, boundaries, goals
├── SOUL.md               # Agent character: voice, principles, identity
├── tools.md              # Command reference (syntax, parsing rules, IDs)
├── README.md             # This file
├── skills/
│   └── contact-find/     # Reusable skill — model-invoked via SKILL.md frontmatter
│       └── SKILL.md
├── .claude/
│   └── settings.local.json  # Claude Code machine-local permissions (gitignored)
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

### 2. Fill in `USER.md` and `SOUL.md`

- **`USER.md`** — Identity, role, communication preferences, timezone, boundaries, current goals, key relationships, decision style.
- **`SOUL.md`** — Agent name, voice, principles, red lines, relationship to the user. Edit only if you want a different *agent*.

### 3. Personalize `AGENTS.md`

Replace `YOUR_NAME` and `YOUR_TIMEZONE`. Adjust the **Hard Rules** for your operator. Add or remove rows in the **Connected Services** table.

### 4. Cache stable IDs in `tools.md`

The bottom of `tools.md` has an "IDs Reference" block. Run the lookup commands once and paste the IDs in (Archive folder, Planner plan/bucket IDs, frequent contacts' AAD IDs). Skills then run with no extra lookups.

### 5. Pick your harness

```bash
# Claude Code
cd clear_assist && claude

# Nanobot / Hermes — point the harness at this directory; AGENTS.md loads automatically
```

---

## Built-in skills

| Skill | What it does |
|---|---|
| `contact-find` | Resolve email + AAD ID + Teams chat ID for a person via Microsoft Graph |

Skills are model-invoked — say what you want (`find Chris Ho`, `look up Sarah's Teams chat ID`) and the matching skill loads automatically. No slash prefix.

You can also speak naturally for anything else: `triage my inbox`, `what's on my calendar today`, `draft a reply to NAME about TOPIC`, `schedule a meeting with NAME next week`.

---

## Adapting the template for a new agent

The framework is the file layout and the rules in `AGENTS.md`. To repurpose:

1. **Swap profile** — rewrite `USER.md` for the new operator and goals; tweak `SOUL.md` if the voice should change.
2. **Swap tools** — replace the M365 sections in `tools.md` with the services your agent needs (Salesforce, Slack, Notion, Linear, etc.).
3. **Swap skills** — edit or replace folders under `skills/`. Each `skills/<name>/SKILL.md` has frontmatter (`name`, `description`) — the description tells the model when to invoke it.
4. **Update `AGENTS.md` Connected Services list** — match the tool reference.

Hard Rules and writing style stay unless you have a specific reason to change them.

---

## Adding services

To add Google Workspace, Todoist, Salesforce, etc.:

1. Add a bullet to the **Connected Services** list in `AGENTS.md`.
2. Add a section to `tools.md` with command syntax + parsing rules + gotchas.
3. Update or add skills in `skills/<name>/SKILL.md` to use the new service.

Always prefer a CLI over an MCP when both exist (per `AGENTS.md` § Connected Services).

---

## Tips

- **Be specific in `USER.md`** — the agent mirrors that voice and respects those boundaries in every draft.
- **Capture new contacts in `USER.md` § Key relationships** as the `contact-find` skill discovers them.
- **Cache IDs in `tools.md` once.** Skills stay fast.
- **Edit skills freely.** Each `SKILL.md` is plain Markdown with frontmatter — adapt the body to your operating cadence, and tune the `description` to match how you actually phrase requests.
- **Single source of truth.** Edit `AGENTS.md`, not `CLAUDE.md` — the latter just imports the former.
