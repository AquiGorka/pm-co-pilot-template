# Gmail Integration (Inbox Access)

## What it does

Read-only access to Gmail. List recent emails, search with Gmail operators, read full email bodies, and download attachments. Used by the PM thread to monitor incoming mail and find specific conversations.

## Auth

Google OAuth2 via a Desktop app client. Gmail API (readonly scope).

```
OAuth credentials:  gmail/credentials.json (from Google Cloud Console)
Saved token:        gmail/token.json (auto-created after first auth, gitignored)
Scope:              https://www.googleapis.com/auth/gmail.readonly
```

Credentials obtained from Google Cloud Console → APIs & Services → Credentials → OAuth client ID (Desktop app). First run opens a browser for Google sign-in, then saves `token.json` for reuse.

The same Google Cloud project and `credentials.json` can be shared with other Google integrations (Calendar, Drive).

## Common Operations

All commands run from the `gmail/` directory.

### List recent emails

```bash
uv run client.py list 10
```

### Search with Gmail operators

```bash
uv run client.py search "from:example" 5
uv run client.py search "has:attachment newer_than:7d"
uv run client.py search "subject:invoice after:2026/04/01"
```

Supports full [Gmail search syntax](https://support.google.com/mail/answer/7190).

### Read an email body

```bash
uv run client.py read <message_id>
```

### Download an attachment

```bash
uv run client.py download <message_id> <attachment_id> <filename>
```

## Scripts

| Script | Purpose |
|--------|---------|
| `gmail/auth.py` | OAuth2 flow — handles token refresh and first-time browser auth |
| `gmail/client.py` | CLI for all Gmail operations (list, search, read, download) |

## Gotchas

- `token.json` is your logged-in state. Treat it like a password. It's gitignored.
- The token auto-refreshes. If auth breaks, delete `token.json` and re-run to re-authenticate.
- Scope is **readonly** — cannot send, delete, or modify emails.
- Gmail API quota: 250 units/second per user. Normal usage is well within limits.
- HTML-only emails without a text/plain part will show "(no text body found)".
- Attachment downloads go to `gmail/downloads/` which is gitignored.
- The Claude Code MCP Gmail connector was evaluated and did not work (auth flow stuck). Direct OAuth2 is reliable.
