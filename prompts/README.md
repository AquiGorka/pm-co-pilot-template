# Prompts — PM-to-IC Thread Delegation

## How this works

The PM thread does not write code. It plans, tracks, and reviews. When implementation work is needed, the PM thread writes a **prompt** — a complete brief for an IC (Individual Contributor) thread to execute.

### The workflow

1. **PM thread writes a prompt.** The prompt is a markdown file in `prompts/` that describes what needs to be done, with enough context for an IC thread to execute independently.
2. **The user takes the prompt to an IC thread.** Copy-paste or reference the file. The IC thread does the work.
3. **IC thread reports back.** The user relays the outcome to the PM thread.
4. **PM thread updates the progress file.** Only the PM thread (via the user) updates progress. The IC thread never touches these files.
5. **PM thread creates follow-up prompts if needed.** This creates a chain of provenance: prompt → progress → follow-up prompt → progress → ...

### File structure

Each prompt gets a numbered directory:

```
prompts/
  README.md                    # this file
  001-example-task/
    prompt.md                  # the brief for the IC thread
    progress.md                # status and outcomes (updated by PM only)
  002-another-task/
    prompt.md
    progress.md
  003-followup-to-001/
    prompt.md                  # references 001, continues the work
    progress.md
```

### Naming convention

- Directory: `NNN-short-description/` (zero-padded 3-digit number + kebab-case description)
- `prompt.md` — the brief. Written by the PM thread. Contains everything the IC thread needs.
- `progress.md` — status tracking. Created by the PM thread when the prompt is created. Updated ONLY when the user reports back from the IC thread.

### What goes in a prompt

A good prompt includes:

- **Goal:** what the IC thread should accomplish
- **Context:** what already exists, where to find it, what decisions have been made
- **Constraints:** language, framework, architecture choices, things to avoid
- **Deliverables:** specific files, scripts, or outputs expected
- **References:** paths to relevant files the IC thread should read first
- **Success criteria:** how to know the work is done

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
- **Done:** delete the directory. The work is captured in the codebase, not in the prompt.
- **In progress:** extend the prune date by 2 weeks. If extended more than twice, reassess whether the work is still relevant.
- **Not started:** delete. If it hasn't been picked up in 2 weeks, it's either not important or needs to be rewritten with fresh context.
- **Blocked:** escalate. Figure out what's blocking it and either unblock or kill it.

### Why this matters

- **Provenance.** Every piece of implementation work has a written brief and a documented outcome.
- **Separation of concerns.** The PM thread focuses on planning and tracking. IC threads focus on execution.
- **Replayability.** If an IC thread loses context or needs to restart, the prompt file has everything it needs.
- **Follow-up chains.** When work needs iteration, the PM thread creates a new prompt that references the previous one. The chain tells the full story.
