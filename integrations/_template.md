# [Service Name]

## What it does

<!-- Brief description of what this integration provides to the PM workflow. -->

## Auth

<!-- How to authenticate. Include header format, token location, etc. -->

```
Header: Authorization: Bearer [TOKEN]
Token location: .env → SERVICE_API_TOKEN
```

## API Reference

<!-- Key endpoints or operations used by the co-pilot. Include working examples. -->

### Example: [Operation Name]

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://api.example.com/v1/resource" | jq '.data'
```

## Gotchas

<!-- Quirks, rate limits, pagination, auth edge cases, etc. -->

- <!-- e.g., "Rate limited to 100 requests/minute" -->
- <!-- e.g., "Pagination: max 100 results per page, use `start_at` param" -->
