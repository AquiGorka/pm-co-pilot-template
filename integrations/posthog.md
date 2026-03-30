# PostHog Integration (Analytics)

## What it does

UI analytics for web apps. Event tracking, user identification, error tracking. Opt-in by default.

## Auth

### Frontend (event capture)

```javascript
posthog.init('[PROJECT_TOKEN]', { api_host: 'https://us.posthog.com' })
```

Token location: `.env` → `POSTHOG_PROJECT_TOKEN`

### Backend (query API)

```
Authorization: Bearer [PERSONAL_API_KEY]
```

Token location: `.env` → `POSTHOG_API_KEY`

## Config

```
Instance:   https://us.posthog.com  (or https://eu.posthog.com)
Project ID: [PROJECT_ID]
```

## Frontend Integration

### Initialize

```javascript
import posthog from 'posthog-js'

posthog.init('[PROJECT_TOKEN]', {
  api_host: 'https://us.posthog.com',
  autocapture: false,        // explicit events only
  capture_pageview: true,
  capture_pageleave: true,
})
```

### Track events

```javascript
posthog.capture('action_completed', { action_type: 'deploy', target: 'staging' })
```

### Identify users

```javascript
posthog.identify('user-id', { role: 'admin', plan: 'pro' })
```

### Track errors

```javascript
window.addEventListener('error', (event) => {
  posthog.capture('$exception', {
    $exception_message: event.message,
    $exception_source: event.filename,
    $exception_lineno: event.lineno,
  })
})
```

## Backend Query API

### Query events

```bash
curl -s 'https://us.posthog.com/api/projects/[PROJECT_ID]/events/?limit=10' \
  -H "Authorization: Bearer [API_KEY]"
```

### Query persons

```bash
curl -s 'https://us.posthog.com/api/projects/[PROJECT_ID]/persons/?limit=10' \
  -H "Authorization: Bearer [API_KEY]"
```

## Source Maps

For minified production builds, upload source maps so stack traces are readable:

```bash
curl -X POST 'https://us.posthog.com/api/projects/[PROJECT_ID]/source_maps/' \
  -H "Authorization: Bearer [API_KEY]" \
  -F "source_map=@dist/assets/index.js.map" \
  -F "js_url=https://[DOMAIN]/assets/index.js"
```

## Gotchas

- `autocapture: false` is recommended for privacy-sensitive apps. Track only what you need.
- PostHog has US and EU instances. Pick the one matching your data residency requirements.
- Personal API key (for queries) is different from the project token (for capture). Don't mix them.
