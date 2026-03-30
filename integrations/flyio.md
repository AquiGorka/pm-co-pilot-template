# Fly.io Integration (Infrastructure)

## What it does

Deploy monitoring, app status checks, logs, secrets management, and SSH access for apps hosted on Fly.io.

## Auth

### API (REST/GraphQL)

```
Authorization: Bearer [FLY_API_TOKEN]
```

Token location: `.env` → `FLY_API_TOKEN`

### CLI

```bash
fly auth login
```

## Common Operations

### List apps

```bash
curl -s -H "Authorization: Bearer [TOKEN]" \
  'https://api.machines.dev/v1/apps?org_slug=[ORG_SLUG]'
```

### Check app status

```bash
fly status -a [APP_NAME]
```

### View logs

```bash
fly logs -a [APP_NAME]
```

### Manage secrets

```bash
# List secrets (names only, values hidden)
fly secrets list -a [APP_NAME]

# Set a secret (triggers redeploy)
fly secrets set SECRET_NAME=value -a [APP_NAME]
```

### SSH into a running machine

```bash
fly ssh console -a [APP_NAME]
```

### Deploy

```bash
fly deploy -a [APP_NAME]
```

### GraphQL API (advanced)

```bash
curl -s -X POST 'https://api.fly.io/graphql' \
  -H "Authorization: Bearer [TOKEN]" \
  -H 'Content-Type: application/json' \
  -d '{"query": "{ apps { nodes { id name status } } }"}'
```

## Object Storage (Tigris)

If using Tigris (S3-compatible storage on Fly.io), see `integrations/tigris.md`.

## Gotchas

- Setting secrets triggers an automatic redeploy. Bundle secret changes to avoid multiple redeploys.
- The Machines API (`api.machines.dev`) is for machine-level operations. The GraphQL API (`api.fly.io/graphql`) is for org/app-level queries.
- Fly.io tokens can be long-lived or short-lived. Use long-lived tokens for CI/automation, short-lived for interactive use.
