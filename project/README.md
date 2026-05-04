# project/

Long-lived project facts that aren't derivable from the code or `actionables.md`. Things like who's involved, why a workstream exists, key decisions and motivations behind ongoing work, and naming conventions across multiple files.

Read at session start (per `../CLAUDE.md`).

## Format

```yaml
---
name: short identifier
description: one-line summary
type: project
---

The fact or decision (one or two sentences).

**Why:** the motivation or constraint behind it.

**How to apply:** how this should shape suggestions or work.
```

## When to add a new file here

- A new initiative kicks off and the why isn't going to be obvious from `actionables.md` alone.
- A naming or scoping decision spans multiple files (e.g., a project rename where some paths stay legacy).
- A person becomes a recurring collaborator and surrounding context matters (preferred channel, role, history with the user).

## When NOT to put something here

- Transient task → `actionables.md`.
- Behavior rule → `feedback/`.
- Pointer to an external system → `reference/`.
- Already documented in code, integration docs, or `decisions.md`.

## Decay

Project facts go stale faster than feedback rules. When a project ends or pivots, retire the file.
