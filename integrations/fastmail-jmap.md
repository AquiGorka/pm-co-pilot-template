# Fastmail JMAP Integration (Email)

## What it does

Query recent inbox emails (subjects + senders) for daily check-ins.

## Auth

Bearer token in header:

```
Authorization: Bearer [FASTMAIL_API_TOKEN]
```

Token location: `.env` → `FASTMAIL_API_TOKEN` and `FASTMAIL_ACCOUNT_ID`

## Base URL

```
https://api.fastmail.com/jmap/api/
```

**Important:** Use `api.fastmail.com`, not `www.fastmail.com`. The `www` endpoint returns 405. Discover the correct API URL via the session endpoint:

```bash
curl -s -L 'https://api.fastmail.com/.well-known/jmap' \
  -H "Authorization: Bearer [TOKEN]"
```

The `apiUrl` field in the response confirms the correct base URL. The session endpoint returns a 302 redirect, so use `-L` to follow it.

## Get Inbox Mailbox ID

```bash
curl -s -X POST "https://api.fastmail.com/jmap/api/" \
  -H "Authorization: Bearer [TOKEN]" \
  -H "Content-Type: application/json" \
  -d '{
    "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
    "methodCalls": [
      ["Mailbox/get", {
        "accountId": "[ACCOUNT_ID]",
        "properties": ["name", "role"]
      }, "a"]
    ]
  }'
```

Filter the response for `role: "inbox"` to get the mailbox ID. Cache this — it doesn't change.

## Query Recent Inbox

```bash
curl -s -X POST "https://api.fastmail.com/jmap/api/" \
  -H "Authorization: Bearer [TOKEN]" \
  -H "Content-Type: application/json" \
  -d '{
    "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
    "methodCalls": [
      ["Email/query", {
        "accountId": "[ACCOUNT_ID]",
        "filter": { "inMailbox": "[INBOX_ID]" },
        "sort": [{ "property": "receivedAt", "isAscending": false }],
        "limit": 10
      }, "a"],
      ["Email/get", {
        "accountId": "[ACCOUNT_ID]",
        "#ids": { "resultOf": "a", "name": "Email/query", "path": "/ids" },
        "properties": ["subject", "from", "receivedAt"]
      }, "b"]
    ]
  }'
```

## Gotchas

- JMAP gives subjects and senders. Body access depends on the token scope.
- Use `api.fastmail.com`, never `www.fastmail.com`.
- The session endpoint (`/.well-known/jmap`) requires `-L` (follow redirects).
