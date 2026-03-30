# ClickUp Integration (Task Management)

## What it does

Task management — create, update, move, tag, and query tasks. Source of truth for what work exists.

## Auth

API token in header:

```
Authorization: [API_TOKEN]
```

Token location: `.env` → `CLICKUP_API_TOKEN`

## Config

```
API base:   https://api.clickup.com/api/v2
List ID:    [LIST_ID]
User ID:    [USER_ID]
```

## Common Operations

### List tasks by status

```bash
curl -s 'https://api.clickup.com/api/v2/list/[LIST_ID]/task?statuses[]=in%20progress&statuses[]=in%20review&subtasks=true&include_closed=false' \
  -H 'Authorization: [API_TOKEN]'
```

### Create a task

```bash
curl -s -X POST 'https://api.clickup.com/api/v2/list/[LIST_ID]/task' \
  -H 'Authorization: [API_TOKEN]' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "Task title",
    "description": "Task description",
    "assignees": [[USER_ID]],
    "status": "to do",
    "tags": ["feat", "module-tag"]
  }'
```

### Update task status

```bash
curl -s -X PUT 'https://api.clickup.com/api/v2/task/[TASK_ID]' \
  -H 'Authorization: [API_TOKEN]' \
  -H 'Content-Type: application/json' \
  -d '{"status": "complete"}'
```

### Add a tag

```bash
curl -s -X POST 'https://api.clickup.com/api/v2/task/[TASK_ID]/tag/[TAG_NAME]' \
  -H 'Authorization: [API_TOKEN]'
```

### Set a custom field (e.g., milestone)

```bash
curl -s -X POST 'https://api.clickup.com/api/v2/task/[TASK_ID]/field/[FIELD_ID]' \
  -H 'Authorization: [API_TOKEN]' \
  -H 'Content-Type: application/json' \
  -d '{"value": "[OPTION_ID]"}'
```

### Create a subtask

```bash
curl -s -X POST 'https://api.clickup.com/api/v2/list/[LIST_ID]/task' \
  -H 'Authorization: [API_TOKEN]' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "Subtask title",
    "parent": "[PARENT_TASK_ID]"
  }'
```

## Gotchas

- Task IDs are alphanumeric (e.g., `86c8m8bvx`), not numeric.
- Custom short IDs (e.g., `ML-1`) require a paid plan upgrade.
- `subtasks=true` must be passed to include subtasks in list queries.
- To auto-link branches to tasks, include the task ID in the branch name (e.g., `feat/TASK_ID-description`). May need a `CU-` prefix depending on workspace settings.
