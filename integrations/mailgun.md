# Mailgun Integration (Transactional Email)

## What it does

Send transactional emails (OTP codes, notifications, alerts) from your domain.

## Auth

Basic auth with API key:

```
-u 'api:[MAILGUN_API_KEY]'
```

Token location: `.env` → `MAILGUN_API_KEY`

## Config

```
Domain:   [MAILGUN_DOMAIN]
API base: https://api.mailgun.net/v3/[MAILGUN_DOMAIN]
Region:   US (api.mailgun.net) or EU (api.eu.mailgun.net)
```

## DNS Setup

Add these records to your domain's DNS:

| Type | Name | Value | Purpose |
|------|------|-------|---------|
| TXT | `@` | `v=spf1 include:mailgun.org ~all` | SPF — authorize Mailgun to send |
| TXT | Various `k1._domainkey` etc. | DKIM key from Mailgun dashboard | DKIM — sign emails |
| CNAME | `email.[DOMAIN]` | `mailgun.org` | Tracking (optional) |
| MX | `[DOMAIN]` | `mxa.mailgun.org` / `mxb.mailgun.org` | Inbound routing (optional) |

Verify DNS setup:

```bash
curl -s -u 'api:[API_KEY]' \
  'https://api.mailgun.net/v3/domains/[DOMAIN]/verify' -X PUT
```

## Send an Email

```bash
curl -s -u 'api:[API_KEY]' \
  'https://api.mailgun.net/v3/[DOMAIN]/messages' \
  -F from='Service Name <noreply@[DOMAIN]>' \
  -F to='recipient@example.com' \
  -F subject='Your verification code' \
  -F text='Your code is: 123456'
```

## Check Domain Status

```bash
curl -s -u 'api:[API_KEY]' \
  'https://api.mailgun.net/v3/domains/[DOMAIN]'
```

## Gotchas

- DNS propagation can take up to 48 hours. Verify with Mailgun's verify endpoint.
- Free tier has a sending limit (typically 100 emails/day for new domains).
- US and EU have different API endpoints. Use the region matching your Mailgun account.
- SPF and DKIM records are required for deliverability. Without them, emails land in spam.
