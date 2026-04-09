# Airtable Integration (Forms + Structured Data)

## What it does

Programmatic management of Airtable bases. Create tables, define fields, build forms, read responses, and write records via the Airtable Web API and Metadata API. Useful for engagement workflows where someone fills out a structured form and the data needs to land in a queryable place that the co-pilot can read and process.

A common use case: hosting a structured intake form, storing responses in a base, and having the co-pilot read those responses to drive the next steps of a workflow.

## Auth

Personal Access Token (PAT) in header:

```
Authorization: Bearer [PAT]
```

Token location: `.env` → `AIRTABLE_PAT`

PATs are created at https://airtable.com/create/tokens. **Required scopes for full programmatic setup:**

- `data.records:read` — read records
- `data.records:write` — create, update, delete records
- `schema.bases:read` — read base structure (tables, fields)
- `schema.bases:write` — create and modify tables and fields via the metadata API

**Critical:** scope the PAT to the specific base only, not "all current and future bases in all workspaces". The latter is overly broad and a security risk.

## Config

```
API base (data):     https://api.airtable.com/v0/[BASE_ID]
API base (metadata): https://api.airtable.com/v0/meta/bases/[BASE_ID]
Base ID:             [BASE_ID] (format: appXXXXXXXXXXXXXX)
```

The Base ID is visible at the top of any base URL: `airtable.com/[BASE_ID]/...` or in the API documentation page accessible from Help → API documentation inside any base.

## Common Operations

### List tables and fields in a base (metadata)

```bash
curl -s -H "Authorization: Bearer $AIRTABLE_PAT" \
  "https://api.airtable.com/v0/meta/bases/$BASE_ID/tables" | jq '.tables[] | {name, fields: [.fields[].name]}'
```

### Create a new table with fields

```bash
curl -s -X POST "https://api.airtable.com/v0/meta/bases/$BASE_ID/tables" \
  -H "Authorization: Bearer $AIRTABLE_PAT" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Experts",
    "description": "Master list of experts",
    "fields": [
      {"name": "Name", "type": "singleLineText"},
      {"name": "Role", "type": "singleLineText"},
      {"name": "Status", "type": "singleSelect", "options": {"choices": [{"name": "Invited"}, {"name": "Active"}, {"name": "Done"}]}}
    ]
  }'
```

### Add a field to an existing table

```bash
curl -s -X POST "https://api.airtable.com/v0/meta/bases/$BASE_ID/tables/$TABLE_ID/fields" \
  -H "Authorization: Bearer $AIRTABLE_PAT" \
  -H "Content-Type: application/json" \
  -d '{"name": "Notes", "type": "multilineText"}'
```

### Update a field (rename, change description, change options)

```bash
curl -s -X PATCH "https://api.airtable.com/v0/meta/bases/$BASE_ID/tables/$TABLE_ID/fields/$FIELD_ID" \
  -H "Authorization: Bearer $AIRTABLE_PAT" \
  -H "Content-Type: application/json" \
  -d '{"description": "Helper text shown below the field on forms"}'
```

### List records in a table

```bash
curl -s "https://api.airtable.com/v0/$BASE_ID/Experts" \
  -H "Authorization: Bearer $AIRTABLE_PAT" | jq '.records[] | {id, fields}'
```

### Create a record

```bash
curl -s -X POST "https://api.airtable.com/v0/$BASE_ID/Experts" \
  -H "Authorization: Bearer $AIRTABLE_PAT" \
  -H "Content-Type: application/json" \
  -d '{
    "fields": {
      "Name": "Jane Doe",
      "Role": "Sample Role",
      "Status": "Invited"
    }
  }'
```

### Update a record

```bash
curl -s -X PATCH "https://api.airtable.com/v0/$BASE_ID/Experts/$RECORD_ID" \
  -H "Authorization: Bearer $AIRTABLE_PAT" \
  -H "Content-Type: application/json" \
  -d '{"fields": {"Status": "Active"}}'
```

### Delete a record

```bash
curl -s -X DELETE "https://api.airtable.com/v0/$BASE_ID/Experts/$RECORD_ID" \
  -H "Authorization: Bearer $AIRTABLE_PAT"
```

### Filter records

```bash
curl -s -G "https://api.airtable.com/v0/$BASE_ID/Experts" \
  -H "Authorization: Bearer $AIRTABLE_PAT" \
  --data-urlencode "filterByFormula={Status}='Active'" | jq '.records[].fields'
```

## Forms

Airtable forms must be **created and configured in the UI** (the metadata API does not support form view creation as of 2026). The recommended workflow:

1. Create the table and fields programmatically via the metadata API
2. In the Airtable UI, switch the table to Form view (click + next to the view tabs → select Form)
3. Set the form title and description
4. Drag internal fields (like auto-numbered IDs, internal notes) to the "Drag and drop fields here to hide" panel on the left
5. Click "Share form" to get the public URL
6. Save the public URL in your project notes

The form's responses appear as new records in the table, which the co-pilot can then read via the data API.

## Field types reference

Common field types and their `type` strings for the metadata API:

| Type | API string | Notes |
|------|-----------|-------|
| Single line text | `singleLineText` | Default for primary fields |
| Long text | `multilineText` | |
| Single select | `singleSelect` | Requires `options.choices` array |
| Multiple select | `multipleSelects` | Requires `options.choices` array |
| Number | `number` | Requires `options.precision` |
| Date | `date` | Requires `options.dateFormat` |
| Checkbox | `checkbox` | Requires `options.icon` and `options.color` |
| URL | `url` | |
| Email | `email` | |
| Phone | `phoneNumber` | |
| Link to another record | `multipleRecordLinks` | Requires `options.linkedTableId` |
| Created time | `createdTime` | Computed |
| Formula | `formula` | Requires `options.formula` |

**Cannot be created via the metadata API:**

- `autoNumber` — auto-incrementing IDs. Workaround: use `singleLineText` or a `formula` field with `RECORD_ID()`.
- `attachment` — must be configured via UI

## Gotchas

- **Form views cannot be created via the API.** They must be configured in the UI. Build the table and fields with the API, then create the form in the UI.
- **`autoNumber` fields cannot be created via the API.** Use `singleLineText` for the primary field instead, or add `autoNumber` manually in the UI after the table is created.
- **The first field of a table is the "primary" field** and must be a text-like type (`singleLineText`, `multilineText`, `formula`, etc.). It cannot be a select, link, attachment, or checkbox.
- **Cannot delete the last table in a base** via API or UI. Airtable requires at least one table. Workaround: create a new table first, then delete the old one.
- **Cannot delete tables via the API at all.** Table deletion is UI-only. Plan for this when designing setup scripts: use `--rename` strategies if you need to "remove" a table programmatically.
- **Free plan limits:** 1,000 records per base, 1,000 API calls/month, 5 requests/second. Sufficient for 1-3 expert engagements but not for high-volume use cases.
- **Field name collisions:** if a table already exists with default fields (e.g., the auto-created `Table 1` has `Name`, `Notes`, `Assignee`, `Status`, `Attachments`), the API will skip creating fields that match by name but with different options. The cleanest path is to delete the default table before running setup, OR rename it out of the way.
- **Trial workspaces:** new accounts get a 14-day trial of paid features. After the trial expires, some features become unavailable. Forms still work on the free tier.
- **PAT base access caching:** if you grant a PAT access to a new base, the change is immediate but `list-bases` API responses may take a few seconds to reflect the new access.
- **Public form URLs are unauthenticated.** Anyone with the link can submit. Don't share publicly if you want only specific people to respond.
- **Spanish/special characters in field names:** field names with accents or special characters (like `Q1 — Portafolio principal en SE`) work fine in the API. URL-encode them when using table or field names in the URL path.

## Setup script pattern

For repeatable, idempotent setup, use a Python script with `requests` (or `pyairtable` for higher-level abstractions). Pattern:

```python
import os
import requests
from dotenv import load_dotenv

load_dotenv()
PAT = os.environ["AIRTABLE_PAT"]
BASE_ID = os.environ["AIRTABLE_BASE_ID"]
HEADERS = {"Authorization": f"Bearer {PAT}", "Content-Type": "application/json"}

# 1. List existing tables
r = requests.get(f"https://api.airtable.com/v0/meta/bases/{BASE_ID}/tables", headers=HEADERS)
existing_tables = {t["name"]: t for t in r.json()["tables"]}

# 2. Create or update each table in your schema
for table_def in MY_SCHEMA:
    if table_def["name"] not in existing_tables:
        # Create new table
        requests.post(f"https://api.airtable.com/v0/meta/bases/{BASE_ID}/tables", headers=HEADERS, json=table_def)
    else:
        # Add missing fields to existing table
        existing_field_names = {f["name"] for f in existing_tables[table_def["name"]]["fields"]}
        for field in table_def["fields"]:
            if field["name"] not in existing_field_names:
                table_id = existing_tables[table_def["name"]]["id"]
                requests.post(f"https://api.airtable.com/v0/meta/bases/{BASE_ID}/tables/{table_id}/fields", headers=HEADERS, json=field)
```

This pattern is idempotent: running it multiple times converges to the desired state without creating duplicates.

## When to use Airtable vs. alternatives

**Use Airtable when:**

- You need a polished form experience for non-technical respondents
- The data needs to be queryable by both humans (in the UI) and machines (via API)
- You want to avoid building and hosting your own form infrastructure
- Volume is low to moderate (under 1,000 records per base on the free tier)

**Don't use Airtable when:**

- You need fully programmatic form creation (use a custom HTML form or Typeform with API instead)
- You need to delete tables or autoNumber fields programmatically
- You need real-time event-driven workflows (Airtable webhooks exist but have limits)
- You're handling sensitive data that shouldn't live in a third-party SaaS
- You need millions of records (use a real database)
