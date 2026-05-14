@AGENTS.md

## Deployment note — exporting for Clearspace

`AGENTS.md` is kept as a template with placeholders. Before exporting this workspace for live Clearspace use, swap the placeholders for the real values in `.env`:

| Placeholder in `AGENTS.md` | Real value lives in `.env` as |
|---|---|
| `https://<your-project-ref>.supabase.co/rest/v1` | `SUPABASE_URL` + `/rest/v1` |
| `sb_publishable_<your-key>` | `SUPABASE_PUBLISHABLE_KEY` |
| `<SUPABASE_JWT_FROM_ENV_OR_AGENTS_SECRET>` | `SUPABASE_JWT` |

`.env` is gitignored — do not commit the substituted file. Do the swap into a separate exported copy, or inject at runtime.
