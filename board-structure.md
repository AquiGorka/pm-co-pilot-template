# Task Board Structure

## Workspace

<!-- Describe the workspace hierarchy. -->
<!-- e.g., [Company] > [Space] > [Folder] > [List] -->

## Lists

| List | Purpose |
|------|---------|
| **[Main list]** | Active work. All new tasks go here. |

## Statuses

| Status | Meaning |
|--------|---------|
| To Do | Scoped but not started |
| In Progress | Actively being worked on |
| In Review | PR submitted / awaiting review |
| Stuck | Blocked, needs attention |
| Complete | Done (prune periodically) |

## Custom Fields

| Field | Type | Values | Purpose |
|-------|------|--------|---------|
| **Milestone** | Dropdown | [See milestones.md] | Groups tasks by high-level goal |

## Tags

### Type of work

| Tag | Meaning |
|-----|---------|
| `feat` | New feature or capability |
| `fix` | Bug fix |
| `chore` | Maintenance, config, CI/CD, infra |
| `docs` | Documentation |
| `debt` | Tech debt |
| `qa` | Quality assurance |
| `product` | Product definition, research, scoping |
| `infra` | Infrastructure, deploy, hosting |

### Module / component

<!-- Map tags to repos or components. -->

| Tag | Repo / Module |
|-----|---------------|
| [tag] | [repo or component name] |

## Views

| View | Type | Purpose |
|------|------|---------|
| **Tasks** | List | Main task view, manually ordered by priority |
| **Board** | Kanban | Day-to-day: To Do > In Progress > In Review > Stuck > Complete |

## Git Linking

<!-- Document how task IDs are linked to branches/PRs. -->
<!-- e.g., Include the task ID in the branch name: feat/TASK-123-description -->

## Task Conventions

- **Title**: Short, descriptive, imperative form (e.g., "Add tracing to API service")
- **Tags**: At least one type tag + one module tag per task
- **Milestone**: Always set (use Backlog if not tied to a milestone)
- **Subtasks**: Break down tasks when they span multiple steps or modules
- **Assignee**: Set when work begins

## Milestones

<!-- Copy milestone names from milestones.md and track progress here. -->

| Milestone | Description | Status | Tasks |
|-----------|-------------|--------|-------|
| [Milestone 1] | [Goal] | [Done/Active/Planned] | [count] |

## API Quick Reference

<!-- Fill in with your task board's API details. -->

```
API base:     [URL]
Auth header:  [HEADER]: [TOKEN]
List/Board:   [ID]
User ID:      [ID]
```

### Common operations

```
Create task:    POST /task
Update task:    PUT /task/{id}
Add tag:        POST /task/{id}/tag/{tag}
Set field:      POST /task/{id}/field/{field_id}  {"value": "option_id"}
Create subtask: POST /task  {"parent": "parent_id"}
```
