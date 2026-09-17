# Jev Browser (agent plugin)

Marketplace-style plugin that teaches agents to automate websites with **high-level goals** via the Mac `jev-browser` CLI (TypeSafe Jev indexed controls), instead of screenshot computer-use.

## What this plugin contains

| Skill | Purpose |
|-------|---------|
| `jev-browser` | When/how to invoke the CLI and parse JSON |
| `jev-browser-setup` | Install the Mac CLI + env |

## What it does **not** include

- API keys
- A hosted browser
- Guaranteed Mac hardware for importers

Importers need a registered Mac with [jev-browser](https://github.com/usingcolor/jev-browser) installed.

## Install (developers)

1. Install the Mac CLI (see `skills/jev-browser-setup/SKILL.md`).
2. Install this plugin / copy the skills into your agent skill library.
3. Point agents at a machine that has `~/.local/bin/jev-browser`.

## License

MIT
