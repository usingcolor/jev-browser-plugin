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
2. `jev-browser` owns the browser session (dedicated CDP Chrome on port `9333` by default).
3. Each step: DOM snapshot → indexed controls → **TypeSafe Jev** picks operation + target → execute.
4. A text model is used **only** for `TYPE_TEXT` fills (`TEXT_MODEL_API_KEY`).
5. Parse **stdout JSON**; always read `status`, not just the process exit code.

Prefer this over screenshot / computer-use for standard HTML/ARIA multi-step flows.

## Invoke

Shell on a **registered Mac** that has `~/.local/bin/jev-browser` (or `~/Codes/jev-browser/bin/jev-browser`).

JSON file:

```bash
export PATH="$HOME/.local/bin:$PATH"
cat > /tmp/jev-req.json <<'JSON'
{
  "goal": "<natural language goal>",
  "start_url": "https://...",
  "max_steps": 40,
  "fields": {},
  "done_when": "optional success criterion"
}
JSON
jev-browser --json-file /tmp/jev-req.json
```

Flags:

```bash
jev-browser --goal '...' --start-url 'https://...' --max-steps 40
jev-browser --stream --goal '...' --start-url 'https://...'
jev-browser --dry-run --goal '...' --start-url 'https://example.com'
jev-browser --doctor
```

## Output

Final stdout object (stream mode also emits per-step lines):

```json
{
  "status": "DONE | NEED_USER | BLOCKED | ERROR",
  "summary": "human-readable",
  "steps": [],
  "url": "final url"
}
```

| Status | Meaning |
|--------|---------|
| `DONE` | Goal completed |
| `NEED_USER` | Login, captcha, permissions, or other human step |
| `BLOCKED` | Agent stopped / no progress |
| `ERROR` | Harness, env, or browser failure |

Exit codes: `0` for `DONE`, `2` for `NEED_USER`/`BLOCKED`, non-zero for `ERROR`. Still parse JSON.

## Env (never print values)

Default file: `~/.config/jev-browser/.env` (mode `600`). Override with `--env-file`.

| Var | Required | Notes |
|-----|----------|-------|
| `TYPESAFE_API_KEY` | yes | TypeSafe Jev |
| `TEXT_MODEL_API_KEY` | optional | OpenAI-compatible key |
| `TEXT_MODEL_BASE_URL` | optional | e.g. `https://api.openai.com/v1` |
| `TEXT_MODEL` | optional | e.g. `gpt-5.6-luna` |
| `TEXT_MODEL_REASONING` | optional | `none` for OpenAI |
| `JEV_BROWSER_CDP_PORT` | optional | default `9333` |

## Goal tips

- One concrete outcome; put exact strings in `fields`.
- Say when to stop (`done_when`).
- Avoid open-ended “explore the site.”

## Limits

- Login walls need a signed-in automation Chrome profile (or user help → `NEED_USER`).
- Canvas / non-DOM UIs may still need computer-use.
- Keep interactive Chrome light on weaker always-on hosts while automation runs.

If the CLI is missing, run the **jev-browser-setup** skill first.


## Typing (`fields`)

Prefer fields over TEXT_MODEL for all agent runs.

Pass every string to type in `fields`. Keys should resemble control labels (`search`, `q`, `email`, …).

Unresolved fills with no text-model fallback return `NEED_USER` / `NEED_FIELDS` — add fields and retry.


## Optional Apple on-device fallback

Prefer `fields`. On Apple Silicon macOS 27+, `fm serve` can back TYPE_TEXT:

```bash
./scripts/apple-fm-serve.sh   # from jev-browser repo
# TEXT_MODEL_BASE_URL=http://127.0.0.1:1976/v1
# TEXT_MODEL=system
# TEXT_MODEL_API_KEY=local
```
