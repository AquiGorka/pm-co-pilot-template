# Porkbun Integration (DNS Management)

## What it does

DNS record management for project domains. Create, update, and delete DNS records via API.

## Auth

API key + secret key in request body:

```json
{
  "apikey": "[PORKBUN_API_KEY]",
  "secretapikey": "[PORKBUN_SECRET_KEY]"
}
```

Token location: `.env` → `PORKBUN_API_KEY` and `PORKBUN_SECRET_KEY`

## Config

```
API base: https://api.porkbun.com/api/json/v3
Domain:   [DOMAIN]
```

## Common Operations

### List all records

```bash
curl -s -X POST 'https://api.porkbun.com/api/json/v3/dns/retrieve/[DOMAIN]' \
  -H 'Content-Type: application/json' \
  -d '{"apikey": "[API_KEY]", "secretapikey": "[SECRET_KEY]"}'
```

### Create a record

```bash
curl -s -X POST 'https://api.porkbun.com/api/json/v3/dns/create/[DOMAIN]' \
  -H 'Content-Type: application/json' \
  -d '{
    "apikey": "[API_KEY]",
    "secretapikey": "[SECRET_KEY]",
    "type": "CNAME",
    "name": "subdomain",
    "content": "target.example.com",
    "ttl": 600
  }'
```

### Edit a record

```bash
curl -s -X POST 'https://api.porkbun.com/api/json/v3/dns/edit/[DOMAIN]/[RECORD_ID]' \
  -H 'Content-Type: application/json' \
  -d '{
    "apikey": "[API_KEY]",
    "secretapikey": "[SECRET_KEY]",
    "type": "A",
    "name": "subdomain",
    "content": "1.2.3.4",
    "ttl": 600
  }'
```

### Delete a record

```bash
curl -s -X POST 'https://api.porkbun.com/api/json/v3/dns/delete/[DOMAIN]/[RECORD_ID]' \
  -H 'Content-Type: application/json' \
  -d '{"apikey": "[API_KEY]", "secretapikey": "[SECRET_KEY]"}'
```

## Gotchas

- Porkbun uses POST for all operations (including reads and deletes), not GET/DELETE.
- Auth goes in the request body, not headers.
- Record IDs are returned in list/create responses. Cache them if you need to edit/delete later.
- TTL is in seconds. 600 (10 min) is a good default for active development.
