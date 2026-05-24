# Hermes Agent Railway Template

## Architecture

Python/Starlette web server that manages two Hermes subprocesses: the gateway and the built-in dashboard.

- `server.py` — Main server: HTTP handlers, gateway + dashboard process managers, basic auth, .env file management
- `templates/index.html` — Single-page UI with Tailwind CSS + Alpine.js
- Config is stored as a flat `.env` file at `/data/.hermes/.env` (Hermes uses python-dotenv)
- Gateway spawned via `hermes gateway`, dashboard via `hermes dashboard --host 0.0.0.0 --port 9119 --insecure --no-open --skip-build`
- Web UI assets pre-built during Docker image creation (`HERMES_WEB_DIST`)

## Key patterns

- Process lifecycle: gateway + dashboard start/stop/restart via async subprocess, stdout captured to ring buffers
- Secret masking: password fields show first 8 chars + `***`, merge on save preserves masked values
- No direct Hermes Python imports — the server manages the .env file independently
- Auto-start: both gateway and dashboard start on server boot if any provider API key is configured
- Ports: 8080 (management UI, basic auth), 9119 (Hermes dashboard, session-token auth)
