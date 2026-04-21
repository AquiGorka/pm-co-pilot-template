# Pi-hole Integration (Network-Wide Ad Blocking)

## What it does

DNS sinkhole that blocks ads, trackers, and malware domains for every device on the network. Runs on a Raspberry Pi or any Linux device. Provides a web admin dashboard for monitoring DNS queries.

## Auth

Web admin interface:

```
URL: http://<pi-ip>/admin/
Password: set during installation
```

## Common Operations

### Check status

```bash
ssh user@raspberrypi.local "pihole status"
```

### Update blocklists (gravity)

```bash
ssh user@raspberrypi.local "pihole -g"
```

### Tail the query log

```bash
ssh user@raspberrypi.local "pihole -t"
```

### Whitelist a domain

```bash
ssh user@raspberrypi.local "pihole -w example.com"
```

### Blacklist a domain

```bash
ssh user@raspberrypi.local "pihole -b example.com"
```

### Disable temporarily (e.g., 5 minutes)

```bash
ssh user@raspberrypi.local "pihole disable 300"
```

### Re-enable

```bash
ssh user@raspberrypi.local "pihole enable"
```

### Update Pi-hole

```bash
ssh user@raspberrypi.local "pihole -up"
```

## Gotchas

- **Requires a static IP** on the Pi. If the IP changes, all devices lose DNS.
- **Set DNS on the router** (DHCP option) for network-wide coverage, or per-device if no router access.
- **Always configure a fallback DNS** (e.g., 1.1.1.1) on devices so internet works if the Pi is down.
- Pi-hole runs as a DNS server — if it's slow or unresponsive, all browsing stalls.
- Docker and bare-metal installs have different admin paths and update methods.
- Major version upgrades (e.g., v5 → v6) have breaking changes. Test before upgrading.
