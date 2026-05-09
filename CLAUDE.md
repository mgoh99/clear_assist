# CLAUDE.md

Entry point for Claude Code. The canonical agent instructions live in [AGENTS.md](AGENTS.md) so the same file works in Nanobot, Hermes, Codex, and any other harness that follows the AGENTS.md convention.

@AGENTS.md

---

## Claude Code-specific notes

These supplement (don't replace) the rules in `AGENTS.md`.

### Slash commands

Pre-built commands live in `.claude/commands/`. Each is a plain Markdown file — edit freely.

| Command | What it does |
|---|---|
| `/triage_inbox` | One-pass Outlook triage — classify, unsubscribe, archive, draft replies |
| `/contact_find <Name>` | Resolve email + AAD ID + Teams chat ID for a person |
| `/planner_add <Name>: <task>` | Create a Planner task assigned to a teammate |

To add a command: drop a new `.md` file in `.claude/commands/`. The filename becomes the slash name.

### Settings

`.claude/settings.local.json` holds machine-local permissions and overrides. It's gitignored equivalent — do not check it in with sensitive values.

### File reading priority

When starting a session:
1. This file (auto-loaded) — pulls in `AGENTS.md` via `@`-import.
2. `tools.md` — read **before** the first call to any connected service whose syntax isn't already in context (Hard Rule #9 in `AGENTS.md`).
3. `context/about-me.md`, `context/people.md`, `context/goals.md`, `context/scheduling.md` — read on demand per the "When to Access" column in AGENTS.md § Key Information Locations.

### Output

Generated artifacts go in `output/` (gitignored except `.gitkeep`). See AGENTS.md § Output Locations for naming conventions.
