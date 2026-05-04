# feedback/

Durable rules learned from working with the user in this repo. Each file is one rule. Claude reads all of them at session start (per `../CLAUDE.md`).

## Format

```yaml
---
name: short identifier
description: one-line summary
type: feedback
---

Rule itself (one or two sentences).

**Why:** the reason or past incident that produced the rule. Knowing the why lets you judge edge cases instead of blindly following.

**How to apply:** when and where the rule kicks in.
```

## Conventions

- File names are lowercase snake_case. No `feedback_` prefix (the directory implies the type).
- One rule per file. If two rules want the same file, they probably want to be one rule with a clearer body, or two files.
- Keep rules short. The body is for the why, not for elaboration of the rule itself.
- New rules just drop in. No registry update, no `CLAUDE.md` edit.
- Amend by editing the file. Retire by deleting.

## What goes here vs elsewhere

- Behavior rule → `feedback/`.
- Long-lived project context (people, motivations, naming) → `project/`.
- Pointer to an external system → `reference/`.
- The user's profile, family, business entity → `user/`.
