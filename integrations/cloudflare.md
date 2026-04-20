# Cloudflare Integration (DNS, Domain Management & Registrar)

## What it does

Manages DNS records, domain settings, Cloudflare tunnels, and domain registration. All domains use Cloudflare nameservers. New domain purchases go through Cloudflare Registrar (at-cost pricing, no markup).

## Auth

API Token in header:

```
Header: Authorization: Bearer [TOKEN]
Token location: .env → CLOUDFLARE_API_TOKEN
```

Token created at https://dash.cloudflare.com/profile/api-tokens.

Recommended permissions:
- Zone / Zone / Read
- Zone / Zone Settings / Edit
- Zone / DNS / Edit
- Account / Account Settings / Read
- Account / Registrar: Domains / Read
- Account / Registrar: Domains / Edit

Scope: all zones, all accounts.

## Config

```
Account ID: .env → CLOUDFLARE_ACCOUNT_ID (or hardcode)
```

## Common Operations

### DNS

#### Verify token

```bash
curl -s -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  "https://api.cloudflare.com/client/v4/user/tokens/verify"
```

#### List zones

```bash
curl -s -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  "https://api.cloudflare.com/client/v4/zones" | python3 -m json.tool
```

#### List DNS records for a zone

```bash
curl -s -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  "https://api.cloudflare.com/client/v4/zones/ZONE_ID/dns_records" | python3 -m json.tool
```

#### Add a DNS record

```bash
curl -s -X POST -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  "https://api.cloudflare.com/client/v4/zones/ZONE_ID/dns_records" \
  -d '{"type":"A","name":"@","content":"1.2.3.4","ttl":1,"proxied":true}'
```

#### Delete a DNS record

```bash
curl -s -X DELETE -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  "https://api.cloudflare.com/client/v4/zones/ZONE_ID/dns_records/RECORD_ID"
```

### Registrar

#### Check domain availability

```bash
curl -s -X POST -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  "https://api.cloudflare.com/client/v4/accounts/ACCOUNT_ID/registrar/domain-check" \
  -d '{"domains":["example.com","example.dev"]}'
```

#### Register a domain

```bash
curl -s -X POST -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  "https://api.cloudflare.com/client/v4/accounts/ACCOUNT_ID/registrar/registrations" \
  -d '{"domain_name":"example.dev"}'
```

#### List registered domains

```bash
curl -s -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  "https://api.cloudflare.com/client/v4/accounts/ACCOUNT_ID/registrar/domains" | python3 -m json.tool
```

## Gotchas

- `ttl: 1` means "automatic" (Cloudflare picks). Use specific seconds only if needed.
- `proxied: true` routes traffic through Cloudflare (CDN, DDoS protection). Set `false` for records that need direct resolution (e.g., MX, mail servers).
- Free plan: 3 page rules, no rate limiting rules, basic DDoS.
- API rate limit: 1,200 requests per 5 minutes per token.
- **Registrar API is in beta** (launched April 2026). Not all TLDs supported via API.
- **Registrar pricing is at-cost.** No markup. WHOIS privacy free.
- **Domains registered via Cloudflare Registrar must use Cloudflare nameservers.**
- **No transfer via API yet** — dashboard only.
- **No renewal endpoint yet** — auto-renew via dashboard or `PUT /registrar/domains/{domain}`.
