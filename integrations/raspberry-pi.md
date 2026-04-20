# Raspberry Pi (Home Network)

## What it does

Always-on Linux device on the home network. Available for local services, automation, monitoring, or as a relay/proxy.

## Auth

SSH key-based access:

```bash
ssh user@raspberrypi.local
```

Set up via `ssh-copy-id user@raspberrypi.local`, then disable password auth in `/etc/ssh/sshd_config`.

## Common Operations

### Connect

```bash
ssh user@raspberrypi.local
```

### Check system status

```bash
ssh user@raspberrypi.local "uptime && free -h && df -h /"
```

### Run a command

```bash
ssh user@raspberrypi.local "command here"
```

## Recommended services

- **Pi-hole** — network-wide DNS ad blocking (~50MB RAM)
- **Tailscale** — mesh VPN for remote access to home network (~30MB RAM)
- **Health monitors** — cron-based pings to external services with alerts
- **Git repo mirrors** — nightly `git clone --mirror` backups

## Gotchas

- **Limited RAM.** Most Pis have 1-4GB. Not suitable for heavy workloads.
- **SD card storage.** Avoid heavy writes — SD cards degrade. Use for read-mostly workloads.
- **No static IP by default.** Uses DHCP — set a DHCP reservation on the router or configure a static IP on the Pi.
- **mDNS (raspberrypi.local)** works from macOS/Linux without knowing the IP.
