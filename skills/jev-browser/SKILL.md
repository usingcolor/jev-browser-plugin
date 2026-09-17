---
name: jev-browser
description: >-
  Use when an agent must complete a multi-step website task via high-level
  GOAL/SITE/FIELDS. Runs Mac CLI jev-browser (TypeSafe Jev indexed controls)
  instead of screenshot computer-use. Requires a registered Mac with the CLI
  installed.
---

# Jev Browser

Agents do **not** drive clicks or screenshots. Pass a high-level goal; the Mac CLI owns Chrome and returns one JSON verdict.

## Architecture

1. Agent issues **goal** (+ optional `start_url`, `fields`, `done_when`, `max_steps`).
2. `jev-browser` owns the browser session (dedicated CDP Chrome by default).
3. Each step: DOM snapshot → indexed controls → **TypeSafe Jev** picks operation + target → execute.
4. A text model is used **only** for `TYPE_TEXT`.
5. Parse **stdout JSON**; always read `status`, not just the process exit code.

Prefer this over screenshot / computer-use for standard HTML/ARIA multi-step flows.

## Invoke

```bash
export PATH="$HOME/.local/bin:$PATH"

echo '{
  "goal": "<natural language goal>",
  "start_url": "https://...",
  "max_steps": 40,
  "fields": {},
  "done_when": "optional success criterion"
}' | jev-browser
```

Flags: `--goal`, `--start-url`, `--max-steps`, `--fields`, `--done-when`, `--stream`, `--dry-run`, `--doctor`.

Shell on a **registered Mac** that has the CLI. An always-on Mini is ideal; a laptop works too.

## Output

```json
{
  "status": "DONE | NEED_USER | BLOCKED | ERROR",
  "summary": "human-readable",
  "steps": [],
  "url": "final url"
}
```

- `DONE` — success
- `NEED_USER` — login / captcha / 2FA; stop and ask the user
- `BLOCKED` — cannot proceed safely
- `ERROR` — harness/browser failure

## Env (never print values)

| Var | Required | Notes |
|-----|----------|-------|
| `TYPESAFE_API_KEY` | yes | TypeSafe Jev |
| `TEXT_MODEL_API_KEY` | for typing | OpenAI-compatible |
| `TEXT_MODEL_BASE_URL` | recommended | e.g. `https://api.openai.com/v1` |
| `TEXT_MODEL` | recommended | e.g. `gpt-5.6-luna` |
| `TEXT_MODEL_REASONING` | optional | `none` for OpenAI |

Loaded from `~/.config/jev-browser/.env` (mode `600`).

## Goal tips

- One concrete outcome; put exact strings in `fields`.
- Say when to stop (`done_when`).
- Avoid open-ended “explore the site.”

## Limits

- Login walls need a signed-in Chrome profile (or user help).
- Canvas / non-DOM UIs may still need computer-use.
- Keep interactive Chrome light on weaker always-on hosts while automation runs.

If the CLI is missing, run the **jev-browser-setup** skill first.
