# Docker Chrome Remote Debugging Environment

A containerized Chrome browser environment with remote debugging support.

## Features

- Access Chrome desktop via Web GUI (KasmVNC)
- Remote debugging via Chrome DevTools Protocol (CDP)
- Connect MCP chrome-devtools from host machine to Chrome inside container

## Architecture

```
Host:9222 → Docker(9222:9223) → Container:9223(Python Forwarder) → Container:9222(Chrome)
```

### Why Port Forwarder?

Chrome 143+ ignores `--remote-debugging-address=0.0.0.0` and only listens on `127.0.0.1`.
A Python script forwards `0.0.0.0:9223` to `127.0.0.1:9222` to enable external access.

## Project Structure

```
.
├── docker-compose.yml      # Docker Compose configuration
├── .mcp.json               # MCP server config (chrome-devtools connection)
├── .claude/
│   └── settings.local.json # Claude Code local permission settings
└── config/                 # Chrome container persistent config (mounted to /config)
    ├── port-forward.py     # Python TCP port forwarder script
    ├── chrome-debug-profile/  # Chrome user data directory (required for remote debugging)
    └── .config/
        └── openbox/
            └── autostart   # Script that runs on container startup
```

## Ports

| Host Port | Container Port | Purpose |
|-----------|----------------|---------|
| 3000 | 3000 | KasmVNC Web GUI (HTTP) |
| 3001 | 3001 | KasmVNC Web GUI (HTTPS) |
| 9222 | 9223 | Chrome DevTools Protocol |

## Usage

### Start Container

```bash
docker compose up -d
```

### Access Chrome GUI

Open `http://localhost:3000` in your browser.

### Verify Remote Debugging

```bash
curl http://localhost:9222/json/version
curl http://localhost:9222/json/list
```

### MCP chrome-devtools Connection

**Option 1: Use CLI command**

```bash
claude mcp add chrome-devtools -- npx chrome-devtools-mcp@latest --browserUrl http://127.0.0.1:9222
```

**Option 2: Use project config**

Configuration is already set in `.mcp.json`. Restart Claude Code to auto-connect.

## Key Configuration Files

### config/.config/openbox/autostart

Chrome startup script inside the container:
- Starts the port forwarder
- Launches Chrome with remote debugging enabled (`--remote-debugging-port=9222`)
- Uses a separate user data directory (`--user-data-dir=/config/chrome-debug-profile`)

### config/port-forward.py

Python TCP port forwarder: `0.0.0.0:9223` → `127.0.0.1:9222`

## License

MIT
