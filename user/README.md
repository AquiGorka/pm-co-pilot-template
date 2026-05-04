# user/

The user's profile, family, working preferences, and anything that personalizes how Claude should engage. Read at session start (per `../CLAUDE.md`).

## Format

```yaml
---
name: short identifier
description: one-line summary
type: user
---

Bullets or short paragraphs covering the facts.
```

## What goes here

- Legal name vs preferred name, and where each gets used.
- Family, personal dates, anniversaries.
- Business entity (legal info, addresses, tax IDs, registration numbers).
- Working preferences that aren't behavior rules but personalize the work (e.g., "loves setting up machines, doesn't want unsolicited migration checklists").

## What does NOT go here

- Behavior rules → `feedback/`.
- Project context → `project/`.
- Anything that needs to be encrypted at rest. Sensitive credentials and tokens belong in `.env.enc` or equivalent, not here. The repo is private but the rule still holds: secrets stay encrypted; this directory is plaintext markdown.
