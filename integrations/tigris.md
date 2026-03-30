# Tigris Integration (Object Storage)

## What it does

S3-compatible object storage on Fly.io. Used for hosting static frontend builds (SPA deployments) and other file storage.

## Auth

AWS-style credentials:

```
AWS_ACCESS_KEY_ID=[TIGRIS_ACCESS_KEY_ID]
AWS_SECRET_ACCESS_KEY=[TIGRIS_SECRET_ACCESS_KEY]
AWS_ENDPOINT_URL_S3=[TIGRIS_ENDPOINT]
AWS_REGION=auto
```

Token location: `.env` → `TIGRIS_ACCESS_KEY_ID`, `TIGRIS_SECRET_ACCESS_KEY`, `TIGRIS_ENDPOINT`

## Config

```
Endpoint:   https://fly.storage.tigris.dev
Public URL: https://[BUCKET_NAME].fly.storage.tigris.dev/[PATH]
```

## Common Operations

### Create a bucket

```bash
aws s3 mb s3://[BUCKET_NAME] \
  --endpoint-url https://fly.storage.tigris.dev
```

Or via Fly CLI:

```bash
fly storage create -n [BUCKET_NAME] -o [ORG_SLUG]
```

### Upload files

```bash
aws s3 sync ./dist/ s3://[BUCKET_NAME]/ \
  --endpoint-url https://fly.storage.tigris.dev \
  --delete
```

### List bucket contents

```bash
aws s3 ls s3://[BUCKET_NAME]/ \
  --endpoint-url https://fly.storage.tigris.dev
```

### Delete a file

```bash
aws s3 rm s3://[BUCKET_NAME]/[PATH] \
  --endpoint-url https://fly.storage.tigris.dev
```

### GitHub Actions deploy pattern

```yaml
- name: Deploy to Tigris
  env:
    AWS_ACCESS_KEY_ID: ${{ secrets.TIGRIS_ACCESS_KEY_ID }}
    AWS_SECRET_ACCESS_KEY: ${{ secrets.TIGRIS_SECRET_ACCESS_KEY }}
    AWS_ENDPOINT_URL_S3: https://fly.storage.tigris.dev
    AWS_REGION: auto
  run: |
    aws s3 sync ./dist/ s3://[BUCKET_NAME]/ --delete
```

## Gotchas

- Uses standard AWS CLI/SDK — just set the endpoint to Tigris.
- Public access is enabled per-bucket via Tigris dashboard or `fly storage update`.
- `--delete` flag in `s3 sync` removes files from the bucket that don't exist locally. Use with care.
- Bucket names must be globally unique across all Tigris users.
