# CLAUDE.md — Windrose Monitor

Agent guidelines for making code changes to this repository.

## Project Overview

Single-file Python tool (`windrose_monitor.py`) that tails a Windrose dedicated server's `R5.log`, maintains a live player roster, serves a web dashboard (`index.html`), and posts Discord notifications. Standard library only — no pip installs, no external deps.

Key files:
- `windrose_monitor.py` — log parsing, roster state, Discord webhook, HTTP server, JSON API
- `index.html` — browser dashboard; polls `/api/players` every 10 seconds, renders client-side
- `assets/` — static assets served at `/assets/...`
- `player_state.json` — persisted recently-seen players (auto-created at runtime)
- `R5.log` — sample/local server log; do not rely on machine-specific paths in examples
- `tests/test_public_api.py` — unit tests using `unittest`

## Running and Testing

No install step required.

```bash
# Run against the sample log
python windrose_monitor.py --log R5.log --host 127.0.0.1 --port 8080

# Syntax/import smoke check
python -m py_compile windrose_monitor.py

# Run tests
python -m pytest tests/
# or without pytest:
python -m unittest discover tests/
```

All CLI options: `python windrose_monitor.py --help`

## Code Conventions

- **Python 3.8+** standard library only. No third-party imports.
- 4-space indentation. `snake_case` functions/variables, `PascalCase` classes, `UPPER_CASE` constants.
- Type hints on public functions. Keep them consistent with existing style.
- Regex constants at module level (e.g. `ROW_RE`, `LOG_LINE_RE`).
- Threading: `Roster` and `KnownPlayers` use `threading.Lock` internally — always acquire before reading/writing `_players`.
- Small focused helpers for parsing and formatting. Keep functions short.
- Comments only when the why is non-obvious (log format edge cases, thread interaction subtleties).

## API Contract

`GET /api/players` returns:
```json
{
  "online_count": 2,
  "online": [{ "name": "...", "state": "Connected", "time_in_game": "...", "joined_at": "..." }],
  "recently_seen": [{ "name": "...", "state": "Disconnected", "left_at": "..." }],
  "as_of": "2026-04-29T12:00:00+00:00",
  "server_started_at": "2026-04-29T10:00:00+00:00",
  "server_info": {
    "server_name": "My Server",
    "max_players": 10,
    "password_protected": false,
    "invite_code": "95c71762",
    "world_id": "...",
    "deployment_id": "..."
  }
}
```

Private identifiers (`account_id`, `session_id`) must never appear in API output. Public fields are defined in `PUBLIC_PLAYER_FIELDS`. Preserve this contract when changing API output.

## Commit Style

Concise imperative summaries: `Add --webhook CLI arg`, `Fix roster dedup on reserved accounts`. One verb, what changed, short subject. No issue references in the commit message.

## Security

- Never commit real Discord webhook URLs, server logs with real player data, or private identifiers.
- Use `DISCORD_WEBHOOK_URL` env var for secrets, not hardcoded values.
- The `resolve_asset_path` function has path traversal protection — preserve it when modifying asset serving.
