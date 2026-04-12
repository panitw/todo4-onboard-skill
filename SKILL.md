---
name: todo4-onboard
description: "Create a Todo4 account and connect your agent — entirely from chat. No website, no password."
metadata:
  openclaw:
    bins: [curl, jq]
---

# Todo4 Onboarding Skill

Set up a Todo4 account, verify the user's email, and connect this agent — all within the conversation. The user never leaves chat.

## Before You Start

Tell the user:

> I'll help you create a Todo4 account. I'll need your email address and will send a verification code to confirm it.

## Onboarding Flow

Follow these steps **in order**, asking **one question at a time**. Wait for the user's response before proceeding.

### Step 1: Collect Email

Ask:

> What email address would you like to use for your Todo4 account?

Wait for the user to provide their email. Validate that it looks like an email address (contains `@` and a `.` after the `@`). If invalid, say:

> That doesn't look like a valid email. Could you try again?

If the user mentions they already have an account, say:

> Looks like you already have a Todo4 account with that email! Let me connect to it instead.

Then proceed with the same flow below — the API handles existing accounts automatically.

### Step 2: Request Verification Code

Run the registration script:

```bash
scripts/register.sh <email>
```

**On success** (exit code 0), say:

> I've sent a 6-digit verification code to **[email]**. Please check your inbox and paste the code here.

**On error**, see [Error Handling](#error-handling) below.

### Step 3: Verify Code

When the user provides the code, run:

```bash
scripts/verify.sh <email> <code>
```

**On success** (exit code 0), the script outputs JSON with `accessToken` and `refreshToken`. Save the `accessToken` value — you'll need it for the next step. Say:

> Email verified! Your Todo4 account is ready. Let me connect myself as your agent...

**On error**, see [Error Handling](#error-handling) below.

### Step 4: Connect Agent

Run the connection script with the access token from Step 3 and your agent name:

```bash
scripts/connect.sh <accessToken> <your_agent_name>
```

Use your actual agent name (e.g., "Claude", "My Assistant") so the user can identify which agent is connected in their Todo4 dashboard. If omitted, defaults to "OpenClaw".

**On success** (exit code 0), the script automatically:
- Writes the MCP config to `~/.openclaw/mcp_config.json`
- Stores the agent token in `~/.openclaw/.env`

Say:

> Done! I'm now connected to your Todo4 account as your AI agent. Your MCP tools are ready to use.

### Step 5: First Task

Prompt the user to try out the connection:

> Try asking me to create a task — like "Create a task to review the Q2 report by Friday"

After the user's first task request, use the Todo4 MCP tools (e.g., `create_task`) to handle it. This confirms the connection is working.

## Error Handling

Handle these situations during the onboarding flow:

| Situation | Script Exit Code | What to Say |
|-----------|-----------------|-------------|
| Invalid email format | register.sh exits 2 (validation error) | "That doesn't look like a valid email. Could you try again?" |
| User didn't receive code | User says so | "Didn't get the code? I can send it again." Then re-run `scripts/register.sh <email>`. |
| Wrong verification code | verify.sh exits 2 (invalid_or_expired_code) | "That code didn't work. Please double-check and try again." |
| Too many failed attempts | verify.sh exits 2 after multiple tries | "Too many attempts. Let me send a new code." Then re-run `scripts/register.sh <email>`. |
| Rate limited | Any script exits 2 (HTTP 429) | "We've hit a rate limit. Please wait a moment and try again." |
| Agent quota exceeded | connect.sh exits 2 (HTTP 422) | "Your account has reached the maximum number of connected agents. You can manage your agents at todo4.io." |
| Network/server error | Any script exits 1 | "I couldn't reach the Todo4 server. Please check your internet connection and try again." |

## Security Rules

**You MUST follow these rules during the onboarding flow:**

- **Never echo back** the full OTP verification code in your messages. If you need to reference it, say "the code you entered" — not the actual digits.
- **Never display** the access token, refresh token, or agent token in the conversation. These are handled by the scripts internally.
- **Never log or display** the full `mcpConfigSnippet` contents — it contains a bearer token.
- If a script fails with unexpected output, summarize the error message for the user without quoting raw JSON responses that may contain sensitive fields.
