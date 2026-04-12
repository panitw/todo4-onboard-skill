# @todo4/onboard — Todo4 OpenClaw Onboarding Skill

Create a Todo4 account and connect your AI agent — entirely from chat. No website, no password.

## Quick Start

After installing the skill, just say:

> Set me up with Todo4

The agent walks you through:
1. Enter your email
2. Paste the 6-digit verification code from your inbox
3. Agent connects itself automatically

Done in under 60 seconds.

## Installation

### Chat Auto-Install (Recommended)

Paste the GitHub URL into your OpenClaw conversation:

```
https://github.com/todo4/openclaw-onboard
```

OpenClaw auto-installs the skill and makes it available immediately.

### ClawHub Auto-Discovery

With ClawHub enabled, say:

> I want to use Todo4

The agent searches ClawHub, finds `@todo4/onboard`, and installs it automatically.

### Settings UI

Open **Settings > Skills** in OpenClaw and search for `todo4-onboard`.

### Terminal (Manual)

```bash
openclaw install @todo4/onboard
```

## Requirements

- **curl** — for HTTP requests to the Todo4 API
- **jq** — for JSON parsing

Both are declared in the skill metadata and checked by OpenClaw at load time.

## Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `TODO4_API_URL` | No | `https://todo4.io/api/v1` | Todo4 API base URL |

## Security

### API Endpoints Called

This skill calls three Todo4 API endpoints during onboarding:

| Endpoint | Purpose | Data Sent |
|----------|---------|-----------|
| `POST /api/v1/auth/register-passwordless` | Create account | Email address |
| `POST /api/v1/auth/verify-otp` | Verify email with OTP code | Email + 6-digit code |
| `POST /api/v1/auth/agent-connect` | Connect agent to account | Agent name + access token |

### Data Handling

- **Email** is the only user PII transmitted. It is sent to the Todo4 API over HTTPS and never stored by the skill.
- **OTP code** is entered by the user and sent to the API for verification. It is not stored or logged.
- **Agent token** is stored in `~/.openclaw/.env` as `TODO4_AGENT_TOKEN` after successful connection. This is the only persistent secret.
- **MCP config** is written to `~/.openclaw/mcp_config.json` with the agent's connection details.
- No passwords are collected or stored. The entire flow is passwordless.

### What This Skill Does NOT Do

- Does not access your filesystem beyond `~/.openclaw/`
- Does not send data to any third party (only the Todo4 API)
- Does not run background processes
- Does not modify any existing files except `~/.openclaw/.env` and `~/.openclaw/mcp_config.json`

## License

MIT — see [LICENSE.md](./LICENSE.md).

## Author

Todo4 / Panit Wechasil
