# Workflows

Repeatable processes that use one or more integrations. Each workflow has a frequency and tracks when it was last run.

Workflows are different from integrations: integrations document *access* to a service (auth, API, common operations). Workflows document *processes* that use those integrations (step-by-step, classification rules, storage paths).

## How it works

Each workflow file has frontmatter:

```yaml
---
frequency: weekly  # daily, weekly, monthly, on-demand
last_run: 2026-04-21 10:30
---
```

During "what's on my plate?" the PM thread checks workflows:
- `daily` — flag if `last_run` is not today
- `weekly` — flag if `last_run` is older than 7 days
- `monthly` — flag if `last_run` is older than 30 days
- `on-demand` — never flag automatically, only runs when the user asks

After running a workflow, update `last_run` with the current timestamp.

## Template

```yaml
---
frequency: weekly
last_run: YYYY-MM-DD HH:MM
---

# Workflow Name

## What it does

One-line description.

## Dependencies

- Integration 1 (`path/`)
- Integration 2 (`path/`)

## Steps

1. Step one
2. Step two
3. ...

## After running

- What to do after the workflow completes (label emails, update records, etc.)
```
