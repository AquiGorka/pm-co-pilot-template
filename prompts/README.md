# Prompts — PM-to-IC Thread Delegation

## How this works

The PM thread does not write code. It plans, tracks, and reviews. When implementation work is needed, the PM thread writes a **prompt** — a complete brief for an IC (Individual Contributor) thread to execute.

### The workflow

1. **PM thread writes a prompt.** The prompt is a markdown file in `prompts/` that describes what needs to be done, with enough context for an IC thread to execute independently.
2. **The user takes the prompt to an IC thread.** Copy-paste or reference the file. The IC thread does the work.
3. **IC thread reports back.** The user relays the outcome to the PM thread.
4. **PM thread updates the progress file.** Only the PM thread (via the user) updates progress. The IC thread never touches these files.
5. **PM thread creates follow-up prompts if needed.** This creates a chain of provenance: prompt → progress → follow-up prompt → progress → ...

### File naming convention

Flat files. No directories. Format: `things-to-do-[sequence]-{prompt or progress}.md`

```
prompts/
  README.md
  things-to-do-1-prompt.md       # first prompt in the chain
  things-to-do-1-progress.md     # progress tracking for prompt 1
  things-to-do-2-prompt.md       # follow-up prompt
  things-to-do-2-progress.md     # progress tracking for prompt 2
```

- `things-to-do` — kebab-case description of the work (stays the same across the chain)
- `[sequence]` — incrementing number for follow-ups in the same chain
- `prompt` or `progress` — the file type

Follow-ups in the same chain keep the same `things-to-do` prefix. This makes it easy to see the full history of a piece of work.

### Working directory rule

IC threads NEVER work in the original repo. Every prompt must instruct the IC thread to:

1. **Clone the repo to `/tmp/`** at the start (e.g., `git clone <repo> /tmp/<repo-name>-<task>`)
2. **Do all work in the `/tmp/` clone**
3. **Push from the `/tmp/` clone** when done (or open a PR)
4. **Do NOT delete the `/tmp/` clone themselves** — cleanup is handled by a separate final prompt

This protects the original working directory from half-finished work, broken state, or accidental file changes.

Every prompt chain should end with a **cleanup prompt** that removes all `/tmp/` clones used during the chain. The PM thread creates this as the final prompt in the sequence (e.g., `things-to-do-3-prompt.md` if the chain had 2 work prompts).

Cleanup prompt template:

```markdown
# Prompt: Cleanup /tmp repos

## Goal
Remove all temporary clones created during this prompt chain.

## Steps
rm -rf /tmp/<repo-name>-<task>

## Success criteria
The /tmp directories no longer exist.
```

### What goes in a prompt

A good prompt includes:

- **Goal:** what the IC thread should accomplish
- **Context:** what already exists, where to find it, what decisions have been made
- **Constraints:** language, framework, architecture choices, things to avoid
- **Working directory:** the `/tmp/` clone path to use (always `/tmp/`, never the original repo)
- **Deliverables:** specific files, scripts, or outputs expected
- **References:** paths to relevant files the IC thread should read first
- **Success criteria:** how to know the work is done
- **Verify before pushing:** exact test commands to run locally before opening a PR

### What goes in a progress file

```markdown
# Progress — [short description]

## Status: [not started | in progress | done | blocked]
## Created: YYYY-MM-DD
## Prune after: YYYY-MM-DD (2 weeks from creation)

## Log

| Date | Event |
|------|-------|
| YYYY-MM-DD | Prompt created. Sent to IC thread. |
| YYYY-MM-DD | IC thread reported: [summary]. [Link to follow-up if any]. |
```

### Pruning

Prompts are pruned every 2 weeks. On the prune date:
- **Done:** delete both files. The work is captured in the codebase, not in the prompt.
- **In progress:** extend the prune date by 2 weeks. If extended more than twice, reassess whether the work is still relevant.
- **Not started:** delete. If it hasn't been picked up in 2 weeks, it's either not important or needs to be rewritten with fresh context.
- **Blocked:** escalate. Figure out what's blocking it and either unblock or kill it.

### Why this matters

- **Provenance.** Every piece of implementation work has a written brief and a documented outcome.
- **Separation of concerns.** The PM thread focuses on planning and tracking. IC threads focus on execution.
- **Replayability.** If an IC thread loses context or needs to restart, the prompt file has everything it needs.
- **Follow-up chains.** When work needs iteration, the PM thread creates a new prompt with the next sequence number. The chain tells the full story.
