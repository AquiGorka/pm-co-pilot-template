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
| **board-structure.md** | Task board structure, tags, milestones, API reference | Rarely |
| **decisions.md** | Decision log | When key decisions are made |
| **weekly-update-format.md** | Template for weekly stakeholder updates | Rarely |
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

## Task Board & Source Control

See `board-structure.md` for full task board setup (statuses, tags, milestones, API reference, git linking) and `integrations/github.md` for source control operations.

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
- [ ] Create initial files: actionables.md, thread.md, system-overview.md, personas.md
- [ ] Customize this file (README.md / copilot-guide) with project name and integration table
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
