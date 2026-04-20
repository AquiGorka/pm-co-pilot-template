# Google Calendar Integration

## What it does

Read events, check availability, create events, and manage scheduling. Used by the PM thread for daily briefings and scheduling.

## Auth

- **Method**: OAuth2 via Google Cloud project (Desktop app credentials)
- **Credentials**: `google-calendar/credentials.json` (OAuth client secret, gitignored)
- **Token**: `google-calendar/token.json` (auto-refreshed, gitignored)
- **Scopes**: `https://www.googleapis.com/auth/calendar` (full read/write)

To re-authenticate (token expired or revoked):

```bash
cd google-calendar && uv run python auth.py
```

The same Google Cloud project and `credentials.json` can be shared with other Google integrations (Gmail, Drive).

## Common Operations

All commands run from `google-calendar/`.

### List today's events

```bash
uv run python cal.py list
```

### List next N days

```bash
uv run python cal.py list 7
```

### Create a timed event

```bash
uv run python cal.py create "Meeting with Alex" "2026-04-21T10:00:00"
```

Defaults to 1 hour. To specify end time:

```bash
uv run python cal.py create "Meeting with Alex" "2026-04-21T10:00:00" "2026-04-21T11:30:00"
```

### Create an all-day event

```bash
uv run python cal.py create "Deadline: proposal" "2026-04-25"
```

### Delete an event

```bash
uv run python cal.py delete <event_id>
```

## Scripts

| Script | Purpose |
|--------|---------|
| `google-calendar/auth.py` | OAuth2 flow — handles token refresh and first-time browser auth |
| `google-calendar/cal.py` | CLI: list, create, delete events |

## Gotchas

- Token auto-refreshes but will eventually expire if unused for months — re-run `auth.py`.
- Set the timezone in the script to match your local timezone.
- `list` shows primary calendar only.
- Event IDs are returned on create — save them if you need to delete later.
- The Claude Code MCP Google Calendar connector was evaluated and did not work (auth flow stuck). Direct OAuth2 is reliable.
