# [PROJECT NAME]: rules for Claude Code

This repo carries its own durable rules and facts learned from working with the user. They live under four sibling directories:

- `feedback/` rules about how Claude should behave in this repo.
- `project/` long-lived project facts (people, motivations, naming conventions).
- `reference/` pointers to information in external systems.
- `user/` the user's profile, family, working preferences, business entity.

These take precedence over Claude default behavior wherever they conflict.

**On session start, read every file in `feedback/`, `project/`, `reference/`, and `user/` before responding.** The combined corpus is small enough to fit in context. Treat them as part of the system prompt.

## Format

Each file uses this frontmatter:

```yaml
---
name: short identifier
description: one-line summary used to decide relevance
type: feedback | project | reference | user
---
```

Body has the rule or fact, a `**Why:**` line (the reason or motivation), and a `**How to apply:**` line. Keep the why so future Claude can judge edge cases instead of blindly following.

## Adding, amending, removing

- New rule or fact: drop a new file in the matching directory. No `CLAUDE.md` edit needed.
- Amend: edit the file directly and commit.
- Retire: delete the file. The rule is gone for future sessions.

Each directory has its own `README.md` documenting scope and the line between "this belongs here" and "this belongs somewhere else."

## Auto-memory relationship

Claude has a per-project auto-memory directory under `~/.claude/projects/...`. The version-controlled rules in this repo are the source of truth. The auto-memory is reserved for ephemeral, in-flight notes that have not yet hardened into a rule. When something hardens, promote it to the repo and remove it from auto-memory.
