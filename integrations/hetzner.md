# Hetzner Cloud Integration (VPS & Infrastructure)

## What it does

Manages VPS instances for hosting services. Good for small Docker Compose workloads at low cost.

## Auth

API Token in header:

```
Header: Authorization: Bearer [TOKEN]
Token location: .env → HCLOUD_TOKEN
```

Token created at https://console.hetzner.cloud → Security → API Tokens. Read/write permissions.

## CLI

```bash
brew install hcloud
```

Auth: set `HCLOUD_TOKEN` env var or `hcloud context create`.

## Common Operations

### List servers

```bash
curl -s -H "Authorization: Bearer $HCLOUD_TOKEN" \
  "https://api.hetzner.cloud/v1/servers" | python3 -m json.tool
```

### Create a server

```bash
curl -s -X POST -H "Authorization: Bearer $HCLOUD_TOKEN" \
  -H "Content-Type: application/json" \
  "https://api.hetzner.cloud/v1/servers" \
  -d '{
    "name": "my-server",
    "server_type": "cx22",
    "image": "ubuntu-24.04",
    "location": "ash",
    "ssh_keys": ["SSH_KEY_NAME"]
  }'
```

### List available server types

```bash
curl -s -H "Authorization: Bearer $HCLOUD_TOKEN" \
  "https://api.hetzner.cloud/v1/server_types" | python3 -c "
import sys, json
data = json.load(sys.stdin)
for t in data['server_types']:
    print(f\"{t['name']:10s} {t['cores']} vCPU / {t['memory']}GB RAM / {t['disk']}GB\")
"
```

### Manage SSH keys

```bash
# List
curl -s -H "Authorization: Bearer $HCLOUD_TOKEN" \
  "https://api.hetzner.cloud/v1/ssh_keys"

# Add
curl -s -X POST -H "Authorization: Bearer $HCLOUD_TOKEN" \
  -H "Content-Type: application/json" \
  "https://api.hetzner.cloud/v1/ssh_keys" \
  -d '{"name":"my-key","public_key":"ssh-ed25519 AAAA..."}'
```

### Create a firewall

```bash
curl -s -X POST -H "Authorization: Bearer $HCLOUD_TOKEN" \
  -H "Content-Type: application/json" \
  "https://api.hetzner.cloud/v1/firewalls" \
  -d '{
    "name": "ssh-only",
    "rules": [{"direction":"in","protocol":"tcp","port":"22","source_ips":["0.0.0.0/0","::/0"]}]
  }'
```

## Gotchas

- Locations: `ash` (Ashburn, US), `nbg1` (Nuremberg, DE), `fsn1` (Falkenstein, DE), `hel1` (Helsinki, FI).
- CX22 is the sweet spot for small Docker Compose workloads: 2 vCPU / 4GB RAM / 40GB disk at ~€4.35/mo.
- Servers are billed hourly. Delete when not needed.
- SSH key must be added before server creation — can't add after.
- Also offers S3-compatible Object Storage (€4.99/mo for 1TB) if needed.
