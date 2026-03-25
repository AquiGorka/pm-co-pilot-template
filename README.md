# PM Co-Pilot Guide

How this PM thread works, what it does, and how to resume after a context loss.

## What This Is

A persistent CLI thread (Claude Code) acting as a PM co-pilot for **[PROJECT NAME]**. It manages tasks, checks integrations, does research, writes documentation, and tracks project state — all conversationally.

**This thread is PM only. No hands-on coding.**

## Integrations

All tokens live in `[PATH_TO_ENV]`. Do NOT `source` this file (special characters may break it). Use inline token values in API calls.

Each integration has its own reference doc in `integrations/` with auth details, API examples, and gotchas. See `integrations/_template.md` for the format.

<!-- Add/remove rows as needed. Common integrations: -->

| Service | What it does | Auth method | Reference |
|---------|-------------|-------------|-----------|
| **[Project Management Tool]** | Task management (create, update, move, tag) | API token in header | `integrations/[tool].md` |
| **[Email]** | Inbox checks for daily check-ins | Bearer token / Basic auth | `integrations/[email].md` |
| **[Calendar]** | Today/tomorrow events for check-ins | Basic auth / OAuth | `integrations/[calendar].md` |
| **[Infrastructure]** | Deploy monitoring | Bearer token | `integrations/[infra].md` |
| **[Observability]** | Traces, metrics, error tracking | Bearer token | `integrations/[observability].md` |
| **[Chat/DMs]** | Team communication, mentions | Bot token | `integrations/[chat].md` |
| **[GitHub]** | PR checks via `gh` CLI | Authenticated via CLI | `integrations/github.md` |

## Daily Check-In

When the user asks "what's on my plate?" (or similar), pull and render (tomorrow first, then today for terminal visibility):

1. **Calendar** — today + tomorrow events (expand recurring)
2. **Inbox** — recent emails relevant to the project
3. **Board** — task board state: in progress, in review, blockers
4. **GitHub PRs** — open PRs authored by user + PRs requesting review
5. **Chat** — mentions and relevant activity (if integration is live)
6. **Reminders** — check actionables against current date
7. **On Mondays** — remind to prune completed items older than a week from local docs

## Key Rules

<!-- Adapt these to the user's preferences. These are learned from feedback, not assumed. -->

- **Do NOT mark tasks complete without asking.** User stays in control.
- **Always validate before taking task board actions** unless told otherwise.
- **Weekly updates**: concise, no padding, no LLM tells. User reviews before posting.

<!-- Add project-specific rules as they emerge from user feedback. -->

## Local Files

All in `[PATH_TO_DOCS]/`:

| File / Directory | Purpose | Update frequency |
|------------------|---------|-----------------|
| **copilot-guide.md** | This file. How the PM thread works. | When process changes |
| **actionables.md** | Active priorities, reminders, people, pending actions | Every session |
| **thread.md** | Running log of key outcomes per date | Every session |
| **system-overview.md** | Architecture, key concepts, competitive landscape | When new understanding emerges |
| **personas.md** | Personas, GTM, market context | When strategy info changes |
| **workstreams.md** | Current state of all active lines of work | When workstream status changes |
| **[board]-structure.md** | Board structure, tags, milestones, conventions | Rarely |
| **decisions.md** | Decision log | When key decisions are made |
| **milestones.md** | Milestone framework (PoC → Production) | When milestones evolve |
| **commits.md** | Commit conventions | Rarely |
| **integrations/** | Per-service integration reference docs | When integrations change |
| **skills/** | Operational prompts (PR review, PR comments, etc.) | When workflows change |

<!-- Add/remove files as needed for the project. -->

## Skills

Reusable prompts for specific operational tasks, stored in `skills/`:

| Skill | Purpose |
|-------|---------|
| **skills/pr-review.md** | Structured code review — finds problems, no opinions |
| **skills/pr-comment.md** | How to post review findings as inline GitHub PR comments |

<!-- Add skills as workflows emerge. Coding thread prompts (briefs for implementation threads) can also live here. -->

## Secrets Management

The `.env` file is gitignored. An encrypted copy (`.env.enc`) is committed using AES-256-CBC via OpenSSL.

**Decrypt (on a fresh clone):**
```bash
openssl enc -aes-256-cbc -d -salt -pbkdf2 -in .env.enc -out .env
```

**Re-encrypt (after editing .env):**
```bash
openssl enc -aes-256-cbc -salt -pbkdf2 -in .env -out .env.enc
```

Both commands prompt for the passphrase.

## Task Board Quick Reference

<!-- Fill in with your project management tool's API details. -->

```
API base:     [URL]
Auth header:  [HEADER_NAME]: [TOKEN]
List/Board:   [ID]
User ID:      [ID]
```

**Custom fields:**
<!-- e.g., Milestone dropdown, Sprint, Priority -->

| Field | ID | Options |
|-------|----|---------|
| [Field name] | [field_id] | [option: id, option: id, ...] |

**Common API patterns:**
```
Create task:    POST /task
Update task:    PUT /task/{id}
Add tag:        POST /task/{id}/tag/{tag}
Set field:      POST /task/{id}/field/{field_id}  {"value": "option_id"}
Create subtask: POST /task  {"parent": "parent_id"}
```

## Source Control Quick Reference

```
Org/Group:  [NAME]
User:       [USERNAME]
Repos:      [list repos]
```

## Resuming After Context Loss

1. Read this file first (copilot-guide.md).
2. Read `actionables.md` for current priorities, reminders, and people.
3. Read `thread.md` for recent session outcomes.
4. Check task board state (in progress / in review).
5. Ask the user what they're working on.

## What We Track Where

- **Task board**: All work items, status, assignments, milestones, tags. Source of truth for "what work exists."
- **Local docs**: Context, strategy, architecture, people notes, GTM, competitive intel, research findings. Source of truth for "why we're doing what we're doing."
- **Both must stay in sync.** Update local docs when tasks are created/completed. Update the board when research produces actionable tasks.

## Setup Checklist (for new projects)

- [ ] Create local docs directory and initialize git repo
- [ ] Set up `.gitignore` and `.env` with integration tokens
- [ ] Encrypt `.env` and commit `.env.enc`
- [ ] Create initial files: copilot-guide.md, actionables.md, thread.md, system-overview.md
- [ ] Set up `integrations/` with a doc per service (use `_template.md`)
- [ ] Set up `skills/` with operational prompts (pr-review, pr-comment)
- [ ] Customize `commits.md` with project conventions
- [ ] Review `milestones.md` and adapt milestone names/criteria to your project
- [ ] Connect to task board: verify API access, list IDs, user IDs
- [ ] Do first check-in to verify all integrations work
- [ ] Capture team members in actionables.md
- [ ] Document board structure (statuses, tags, milestones)
- [ ] Set up recurring reminders (meetings, check-ins)
