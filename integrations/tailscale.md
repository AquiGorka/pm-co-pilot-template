# Tailscale Integration (Mesh VPN & Remote Access)

## What it does

WireGuard-based mesh VPN connecting devices across networks. A Raspberry Pi (or any always-on device) can act as a subnet router, exposing an entire home/office network to the tailnet. Enables remote access to local machines from anywhere.

## Auth

API Key in Basic auth header:

```
Authorization: Basic $(echo -n "$TAILSCALE_API_KEY:" | base64)
Token location: .env → TAILSCALE_API_KEY
```

Key created at https://login.tailscale.com/admin/settings/keys. **Expires every 90 days max** — must be regenerated. Set a recurring reminder.

OAuth clients (non-expiring) require a paid plan (Starter+).

Admin console: https://login.tailscale.com/admin

## Common Operations

### List devices

```bash
source .env; curl -s -u "$TAILSCALE_API_KEY:" \
  "https://api.tailscale.com/api/v2/tailnet/-/devices" | python3 -c "
import sys, json
for d in json.load(sys.stdin)['devices']:
    print(f\"{d['hostname']:25s} {d.get('addresses',[''])[0]:20s} online={d.get('online',False)}\")
"
```

### Get device details

```bash
source .env; curl -s -u "$TAILSCALE_API_KEY:" \
  "https://api.tailscale.com/api/v2/device/DEVICE_ID" | python3 -m json.tool
```

### List subnet routes

```bash
source .env; curl -s -u "$TAILSCALE_API_KEY:" \
  "https://api.tailscale.com/api/v2/device/DEVICE_ID/routes" | python3 -m json.tool
```

### Approve a subnet route

```bash
source .env; curl -s -X POST -u "$TAILSCALE_API_KEY:" \
  -H "Content-Type: application/json" \
  "https://api.tailscale.com/api/v2/device/DEVICE_ID/routes" \
  -d '{"routes":["192.168.1.0/24"]}'
```

### Get DNS settings

```bash
source .env; curl -s -u "$TAILSCALE_API_KEY:" \
  "https://api.tailscale.com/api/v2/tailnet/-/dns/nameservers"
```

### Check tailnet status (CLI, local device only)

```bash
tailscale status
```

## Gotchas

- **API keys expire at 90 days max.** Cannot create non-expiring keys on the free plan. Set a recurring reminder.
- **API keys inherit creator permissions.** If the user loses access, the key stops working.
- **The `tailscale` CLI is local-only.** It talks to the local daemon, not the cloud API. Cannot remotely manage other nodes via CLI.
- **MagicDNS names are not stable identifiers.** Use device IDs for automation.
- **Free tier:** 3 users, 100 devices. No OAuth clients.
- **Subnet router must have IP forwarding enabled** (`net.ipv4.ip_forward = 1`).
- **Auth keys vs API keys** — different things. Auth keys register new devices onto the tailnet. API keys authenticate REST API calls.
