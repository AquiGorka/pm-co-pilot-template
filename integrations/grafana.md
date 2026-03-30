# Grafana Cloud Integration (Observability)

## What it does

Dashboards, traces, and metrics for monitoring deployed services. Query traces via TraceQL, check dashboard status, and verify telemetry ingestion.

## Auth

Service account Bearer token:

```
Authorization: Bearer [GRAFANA_SA_TOKEN]
```

Token location: `.env` → `GRAFANA_SA_TOKEN`

Instance URL: `[INSTANCE].grafana.net`

## OTLP Ingestion

For services sending traces/metrics via OpenTelemetry:

```
Endpoint: https://otlp-gateway-[REGION].grafana.net/otlp
Auth header: Authorization: Basic [GRAFANA_OTLP_AUTH]
```

Token location: `.env` → `GRAFANA_OTLP_AUTH`

## Common Operations

### List dashboards

```bash
curl -s -H "Authorization: Bearer [SA_TOKEN]" \
  'https://[INSTANCE].grafana.net/api/search?type=dash-db'
```

### Get organization info

```bash
curl -s -H "Authorization: Bearer [SA_TOKEN]" \
  'https://[INSTANCE].grafana.net/api/org'
```

### Query traces (Tempo via TraceQL)

```bash
curl -s -H "Authorization: Basic [TEMPO_AUTH]" \
  'https://[INSTANCE].grafana.net/api/datasources/proxy/[TEMPO_DS_UID]/api/search?q=%7B%7D&limit=10'
```

### TraceQL search examples

```
{}                           # all traces
{span.http.method = "POST"} # POST requests only
{status = error}             # error traces
{resource.service.name = "[SERVICE]"} # by service
```

## Gotchas

- Service account tokens and OTLP ingestion tokens are different. SA tokens query the API; OTLP tokens push telemetry.
- Tempo (traces) auth uses Basic auth, not Bearer. The token is a base64-encoded `user:key` pair.
- Rate limits apply on the free tier. Check `X-RateLimit-*` headers.
