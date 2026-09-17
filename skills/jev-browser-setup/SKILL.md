---
name: jev-browser-setup
description: >-
  Use when installing or repairing the Mac jev-browser CLI (uv, Chrome CDP,
  TypeSafe/OpenAI env) so agents can run the jev-browser skill.
---

# Jev Browser setup

Install the Mac worker that the **jev-browser** skill calls.

## Requirements

- macOS with Google Chrome
- `uv` (https://docs.astral.sh/uv/)
- Network access to TypeSafe + your text-model provider
- A Grok Bot / Cursor **registered machine** for Shell with `machineId`

## Steps

1. Install `uv` if needed: `curl -LsSf https://astral.sh/uv/install.sh | sh`
2. Clone CLI + upstream (paths are conventional):
   - `~/Codes/jev-browser` — https://github.com/usingcolor/jev-browser
   - `~/Codes/jev-ultrafast` — https://github.com/browser-use/jev-ultrafast (or the fork your install docs name)
3. Apply patches from `jev-browser/patches/` onto `jev-ultrafast` when present.
4. `cd ~/Codes/jev-ultrafast && uv sync`
5. `cd ~/Codes/jev-browser && uv sync`
6. Symlink: `ln -sfn ~/Codes/jev-browser/bin/jev-browser ~/.local/bin/jev-browser`
7. Create `~/.config/jev-browser/.env` (chmod 600) with at least:
   - `TYPESAFE_API_KEY=...`
   - `TEXT_MODEL_API_KEY=...`
   - `TEXT_MODEL_BASE_URL=https://api.openai.com/v1`
   - `TEXT_MODEL=gpt-5.6-luna`
   - `TEXT_MODEL_REASONING=none`
8. Never print secret values. Confirm with `jev-browser --doctor` (booleans only).
9. Smoke: `jev-browser --goal 'Open example.com. DONE when Example Domain is visible.' --start-url https://example.com --max-steps 8`

## Notes

- Prefer an always-on Mac Mini for shared agent use; Amphetamine/sleep policy helps.
- If Apple CLT/`xcrun` is broken, use Homebrew `git` and uv-managed Python (the launcher prefers `.venv`).
- Sites like X may require a one-time login in the automation Chrome profile.
