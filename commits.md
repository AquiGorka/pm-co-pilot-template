# Commit Conventions

## Rules

- **Semantic commits.** Use conventional commit prefixes: `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, `style:`, `test:`.
- **Atomic commits.** Each commit should do one thing. Don't mix unrelated changes.
- **Sequential/incremental order.** Commits should tell a logical story — each building on the previous one in a coherent sequence.

<!-- Add project-specific rules as needed. Examples: -->
<!-- - No co-authorship trailers. -->
<!-- - Version bump must be the last commit in a PR. -->

## Examples

```
docs: add integration references for ClickUp and Fastmail
chore: encrypt .env and gitignore plaintext secrets
feat: add onboarding wizard to council console
fix: handle empty response from channel state query
```
