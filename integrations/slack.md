# Slack Integration

## What it does

Read and send messages, search conversations, list channels, and monitor activity across Slack workspaces.

## Approaches (ranked by effort)

### 1. MCP Server with session tokens (recommended — no app creation)

**korotovsky/slack-mcp-server** — most popular option. Supports "stealth mode" using browser session tokens (`xoxc-`/`xoxd-`). No Slack app creation, no admin approval, no OAuth.

- 15 tools: search, messages, DMs, user groups, channels
- Read-only by default (write can be enabled)

**Setup:**

```bash
# You'll need xoxc- token + xoxd- cookie from browser:
# 1. Open Slack in browser
# 2. DevTools → Network → filter "api" → grab token from any request body
# 3. DevTools → Application → Cookies → copy "d" cookie value
```

Repo: https://github.com/korotovsky/slack-mcp-server

### 2. Bot token + custom scripts (traditional)

Create a Slack app at api.slack.com, install per workspace, use `xoxb-` bot tokens.

**Required scopes:** `channels:read`, `channels:history`, `groups:read`, `groups:history`, `im:read`, `im:history`, `chat:write`, `users:read`

## Auth

| Approach | Token type | Per-workspace? | Needs admin? | Longevity |
|----------|-----------|:-:|:-:|-----------|
| Session tokens (MCP stealth) | `xoxc-` + `xoxd-` cookie | Yes | No | Weeks/months, rotates on logout |
| Bot token (app) | `xoxb-` | Yes | Maybe | Stable until revoked |

```
Token location: .env → SLACK_TOKEN_<WORKSPACE_NAME>=xoxb-...  (bot token approach)
                .env → SLACK_XOXC_<WORKSPACE_NAME>=xoxc-...   (session token approach)
                .env → SLACK_XOXD_<WORKSPACE_NAME>=xoxd-...   (session cookie)
```

## Common Operations (bot token / curl)

### List channels

```bash
curl -s -H "Authorization: Bearer $SLACK_BOT_TOKEN" \
  "https://slack.com/api/conversations.list?types=public_channel,private_channel&limit=100" | jq '.channels[] | {id, name}'
```

### Read recent messages

```bash
curl -s -H "Authorization: Bearer $SLACK_BOT_TOKEN" \
  "https://slack.com/api/conversations.history?channel=CHANNEL_ID&limit=20" | jq '.messages[] | {user, text, ts}'
```

### Send a message

```bash
curl -s -X POST -H "Authorization: Bearer $SLACK_BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"channel":"CHANNEL_ID","text":"Hello"}' \
  "https://slack.com/api/chat.postMessage" | jq '.ok'
```

### Search messages

```bash
curl -s -H "Authorization: Bearer $SLACK_USER_TOKEN" \
  "https://slack.com/api/search.messages?query=keyword&count=10" | jq '.messages.matches[] | {text, channel: .channel.name, ts}'
```

Note: `search.messages` requires a **user token** (`xoxp-`), not a bot token.

## Gotchas

- **No single API key across workspaces.** Each workspace requires its own token.
- **Bot must be invited** to channels to read them. Session tokens don't have this limitation.
- **Session tokens are fragile** — they rotate on logout, password change, or admin revocation.
- **Rate limits:** Tier 1 methods ~1 req/sec. Tier 2 (search) ~20 req/min.
- **`search.messages` needs a user token** (`xoxp-`), not a bot token. Common gotcha.
- **Admin approval** may be needed to install bot apps in workspaces you don't own.
- **MCP approach is the path of least resistance** — integrates directly into Claude Code, no separate scripts needed.
