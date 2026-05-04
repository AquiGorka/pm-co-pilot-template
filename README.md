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

## Rules

The durable rules learned from working with the user live in version-controlled directories at the repo root, auto-loaded via `CLAUDE.md`:

| Directory | What it holds |
|-----------|---------------|
| `feedback/` | Behavior rules. How Claude should act in this repo. |
| `project/` | Long-lived project facts. People, motivations, naming conventions. |
| `reference/` | Pointers to information in external systems. |
| `user/` | The user's profile, family, working preferences, business entity. |

`CLAUDE.md` instructs Claude to read every file in those four directories on session start. Each directory has a `README.md` documenting scope and the line between "this belongs here" and "this belongs elsewhere."

**Adding a rule:** drop a new file in the matching directory. No `CLAUDE.md` edit needed. The file format is YAML frontmatter (`name`, `description`, `type`) plus a body with a `**Why:**` line and a `**How to apply:**` line.

The auto-memory directory at `~/.claude/projects/<repo-id>/memory/` is reserved for ephemeral, in-flight notes that have not yet hardened into a rule. Promote to the repo when something hardens.

## Local Files

All in `[PATH_TO_DOCS]/`:

| File / Directory | Purpose | Update frequency |
|------------------|---------|-----------------|
| **README.md** | This file. How the PM thread works. | When process changes |
| **CLAUDE.md** | Auto-loaded session bootstrap. Directs Claude to read the rules directories. | Rarely |
| **feedback/** | Per-rule files: how Claude should behave. | When a new rule emerges |
| **project/** | Long-lived project facts (people, motivations, naming). | When new initiatives or collaborators appear |
| **reference/** | Pointers to external systems. | When a new external system enters the workflow |
| **user/** | User's profile, family, working preferences, business entity. | Rarely |
| **actionables.md** | Active priorities, reminders, people, pending actions. | Every session |
| **thread.md** | Running log of key outcomes per date. | Every session |
| **workstreams.md** | Current state of all active lines of work. | When workstream status changes |
| **board-structure.md** | Task board structure, tags, milestones, API reference. | Rarely |
| **decisions.md** | Decision log. | When key decisions are made |
| **weekly-update-format.md** | Template for weekly stakeholder updates. | Rarely |
| **milestones.md** | Milestone framework (PoC to Production). | When milestones evolve |
| **commits.md** | Commit conventions. | Rarely |
| **metrics.md** | KPI definitions (throughput, quality, time-to-deliver) computed from `prompts/`. | When the schema evolves |
| **prompts/** | PM-to-IC delegation chain. Each work item has a `*-prompt.md` brief and a `*-progress.md` tracker. See `prompts/README.md`. | Every session |
| **integrations/** | Per-service integration reference docs. | When integrations change |
| **skills/** | Operational prompts (PR review, PR comments, etc.). | When workflows change |
| **workflows/** | Recurring workflows (e.g. monthly email reviews). | When a new recurring task is formalized |
| **daily/** | Per-day daily logs (food, sleep, mood, exercise, work focus, log entries). | Daily |

<!-- Add/remove files as needed for the project. -->

## Skills

Reusable prompts for specific operational tasks, stored in `skills/`:

| Skill | Purpose |
|-------|---------|
| **skills/pr-review.md** | Structured code review — finds problems, no opinions |
| **skills/pr-comment.md** | How to post review findings as inline GitHub PR comments |
| **skills/open-pr-checklist.md** | Pre-flight checklist before opening a PR (tests, git hygiene, metadata, docs) |

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

## Task Board & Source Control

See `board-structure.md` for full task board setup (statuses, tags, milestones, API reference, git linking) and `integrations/github.md` for source control operations.

## Resuming After Context Loss

1. `CLAUDE.md` will already be loaded by Claude Code. Read every file under `feedback/`, `project/`, `reference/`, and `user/` so the rules and facts are in context.
2. Read this file (README.md).
3. Read `actionables.md` for current priorities, reminders, and people.
4. Read `thread.md` for recent session outcomes.
5. Check the task board state (in progress / in review).
6. Ask the user what they're working on.

## What We Track Where

- **Task board**: All work items, status, assignments, milestones, tags. Source of truth for "what work exists."
- **Local docs**: Context, strategy, architecture, people notes, GTM, competitive intel, research findings. Source of truth for "why we're doing what we're doing."
- **Both must stay in sync.** Update local docs when tasks are created/completed. Update the board when research produces actionable tasks.

## Updating from the Template

Projects forked from this template evolve independently. When the template gets new files, patterns, or integrations, specific projects need to manually pull in what's relevant.

### How to check for updates

```bash
# Add the template as a remote (one-time)
git remote add template [TEMPLATE_REPO_URL]

# Fetch the latest template changes
git fetch template main

# See what changed since you last synced
git log template/main --oneline --since="2 weeks ago"

# Diff a specific file against the template version
git diff HEAD template/main -- integrations/_template.md
```

### How to pull in updates

**Do NOT merge or rebase the template into your project.** Your project has customized most files (README, actionables, workstreams, etc.) and a merge would create conflicts everywhere.

Instead, review changes manually and cherry-pick what's useful:

1. Run `git fetch template main` to get the latest template state
2. Review the template's recent commits: `git log template/main --oneline -10`
3. For each commit that looks relevant, inspect the diff: `git show template/main:<filepath>`
4. Copy the parts that apply to your project. Adapt to your project-specific context.

### What typically gets pulled in

- **New integration templates** (`integrations/*.md`) — copy the generic template, then customize with your project's auth details and endpoints
- **New skills** (`skills/*.md`) — usually usable as-is
- **New workflow patterns** (`prompts/README.md`, methodology updates) — review and adopt if they fit
- **README updates** (new sections, improved checklists) — cherry-pick relevant sections

### What typically does NOT get pulled in

- **Files you've already customized** (README, actionables, workstreams, etc.) — your project version is the source of truth, the template version is irrelevant once customized
- **Integration docs you don't use** — the template has many, your project only needs the ones you've set up

### When to check

Add a recurring reminder to check for template updates. Monthly is a good cadence. Or check when you're about to set up a new integration and want to see if the template has a reference doc for it.

## Setup Checklist (for new projects)

- [ ] Create local docs directory and initialize git repo
- [ ] Set up `.gitignore` and `.env` with integration tokens
- [ ] Encrypt `.env` and commit `.env.enc`
- [ ] Customize `README.md` and `CLAUDE.md` with project name and any project-specific bootstrap text
- [ ] Seed `user/` with the user's profile and business entity (or copy from another repo if shared)
- [ ] Drop a starter `feedback/` rule or two if any are obvious upfront; the rest accrete from real interactions
- [ ] Create initial files: actionables.md, thread.md
- [ ] Fill in `board-structure.md` with task board API details, IDs, statuses, tags
- [ ] Fill in `decisions.md` with initial tooling/process decisions
- [ ] Fill in `workstreams.md` with current lines of work
- [ ] Set up `integrations/` with a doc per service (use `_template.md` or copy from included templates)
- [ ] Customize `commits.md` with project conventions
- [ ] Review `milestones.md` and adapt milestone names/criteria to your project
- [ ] Customize `weekly-update-format.md` with your posting channel and project name
- [ ] Connect to task board: verify API access, list IDs, user IDs
- [ ] Do first check-in to verify all integrations work
- [ ] Capture team members in actionables.md
- [ ] Set up recurring reminders (meetings, check-ins)
