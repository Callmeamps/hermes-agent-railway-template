# Hermes Agent Railway Template

One-click deploy [Hermes Agent](https://github.com/nousresearch/hermes-agent) on [Railway](https://railway.app) with a web-based config UI and status dashboard.

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/hermes-agent)

## What you get

- **Web Config UI** — configure LLM providers, messaging channels, tool API keys, and model settings from your browser
- **Status Dashboard** — monitor gateway state, uptime, provider/channel status, and live logs
- **Hermes Dashboard** — full Hermes web UI on port 9119 with session-token auth
- **Gateway Management** — start, stop, and restart the Hermes gateway from the UI
- **Basic Auth** — password-protected admin panel
- **Persistent Storage** — config and data survive container restarts via Railway volume

## Quick Start

### Deploy to Railway

1. Click the "Deploy on Railway" button above
2. Set the `ADMIN_PASSWORD` environment variable (or a random one will be generated and printed to logs)
3. Attach a volume mounted at `/data`
4. Open your app URL — you'll be prompted for credentials (default username: `admin`)
5. Configure at least one LLM provider API key and your messaging channels, then hit Save
6. Once setup is complete, remove the public endpoint from your Railway service — the web UI is only needed for initial configuration and Hermes operates entirely through its configured channels (Telegram, Discord, Slack, etc.)

### Run Locally with Docker

```bash
docker build -t hermes-agent .
docker run --rm -it -p 8080:8080 -p 9119:9119 -e PORT=8080 -e ADMIN_PASSWORD=changeme -v hermes-data:/data hermes-agent
```

Open `http://localhost:8080` and log in with `admin` / `changeme`. The Hermes dashboard is available at `http://localhost:9119`.

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `8080` | Web server port |
| `DASHBOARD_PORT` | `9119` | Hermes dashboard port |
| `ADMIN_USERNAME` | `admin` | Basic auth username |
| `ADMIN_PASSWORD` | *(generated)* | Basic auth password. If unset, a random password is generated and printed to stdout |

All Hermes configuration (LLM providers, messaging channels, tool API keys) is managed through the web UI.

## Architecture

```
Railway Container
├── Python Web Server (Starlette + uvicorn) :8080
│   ├── / — Config editor + status dashboard
│   ├── /health — Health check (no auth)
│   └── /api/* — Config, status, logs, gateway + dashboard control
├── hermes gateway — managed as async subprocess
└── hermes dashboard — FastAPI web UI :9119 (session-token auth)
```

The management server runs on `$PORT` (8080) and manages both the Hermes gateway and dashboard as child processes. Gateway and dashboard stdout/stderr is captured into ring buffers and viewable in the management UI.

## API Endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `GET` | `/` | Yes | Web UI |
| `GET` | `/health` | No | Health check |
| `GET` | `/api/config` | Yes | Get config (secrets masked) |
| `PUT` | `/api/config` | Yes | Save config |
| `GET` | `/api/status` | Yes | Gateway, dashboard, provider, channel status |
| `GET` | `/api/logs` | Yes | Recent gateway log lines |
| `GET` | `/api/dashboard/logs` | Yes | Recent dashboard log lines |
| `POST` | `/api/gateway/start` | Yes | Start gateway |
| `POST` | `/api/gateway/stop` | Yes | Stop gateway |
| `POST` | `/api/gateway/restart` | Yes | Restart gateway |
| `POST` | `/api/dashboard/start` | Yes | Start dashboard |
| `POST` | `/api/dashboard/stop` | Yes | Stop dashboard |
| `POST` | `/api/dashboard/restart` | Yes | Restart dashboard |

## Supported Providers

OpenRouter, DeepSeek, DashScope, GLM/Z.AI, Kimi, MiniMax, Hugging Face

## Supported Channels

Telegram, Discord, Slack, WhatsApp, Email, Mattermost, Matrix
