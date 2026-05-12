# CLAUDE.md

Entry point for Claude Code. The canonical agent instructions live in [AGENTS.md](AGENTS.md) so the same file works in Nanobot, Hermes, Codex, and any other harness that follows the AGENTS.md convention.

@AGENTS.md

---

## Claude Code-specific notes

These supplement (don't replace) the rules in `AGENTS.md`.

### Skills

Reusable capabilities live in `skills/<name>/SKILL.md`. Each skill has frontmatter (`name`, `description`) that Claude reads to decide when to invoke it — no slash command needed, just describe the task and the matching skill loads automatically.

| Skill | What it does |
|---|---|
| `contact-find` | Resolve email + AAD ID + Teams chat ID for a person via Microsoft Graph |

To add a skill: create `skills/<kebab-name>/SKILL.md` with a `description` field that lists the triggers (verbs/phrases the user is likely to say). Skills work across Claude Code, Nanobot, and Hermes — same file, same convention.

### Settings

`.claude/settings.local.json` holds machine-local permissions and overrides. It's gitignored equivalent — do not check it in with sensitive values.

### File reading priority

When starting a session:
1. This file (auto-loaded) — pulls in `AGENTS.md` via `@`-import.
2. `tools.md` — read **before** the first call to any connected service whose syntax isn't already in context (Hard Rule #9 in `AGENTS.md`).
3. `context/about-me.md`, `context/people.md`, `context/goals.md`, `context/scheduling.md` — read on demand per the "When to Access" column in AGENTS.md § Key Information Locations.

### Output

Generated artifacts go in `output/` (gitignored except `.gitkeep`). See AGENTS.md § Output Locations for naming conventions.
