# Open PR Checklist

A pre-flight checklist to run before opening a pull request. The PM co-pilot can run through this with the user to catch common issues before a PR goes up for review.

## When to use

When the user says "I'm about to open a PR" or "review my branch before I push" or similar. Run through the checklist and flag anything that's off.

## Checklist

### Code quality
- [ ] All tests pass locally
- [ ] No debug code left (console.log, print statements, TODO/FIXME that shouldn't ship)
- [ ] No commented-out code blocks
- [ ] No hardcoded secrets, API keys, or credentials
- [ ] Linter/formatter has been run (no style violations)

### Git hygiene
- [ ] Branch is rebased on the latest main (no unnecessary merge commits)
- [ ] Commits are atomic and tell a logical story
- [ ] Commit messages follow the project's conventions (see `commits.md`)
- [ ] No unrelated changes bundled in

### PR metadata
- [ ] PR title is short, descriptive, imperative form (under 70 characters)
- [ ] PR description summarizes what changed and why
- [ ] Linked to the relevant task/issue (if using a task board)
- [ ] Reviewer(s) assigned
- [ ] Labels/tags applied (if the project uses them)

### Deployment and risk
- [ ] No breaking changes without a migration path
- [ ] Environment variables or config changes documented
- [ ] Database migrations included (if applicable)
- [ ] Feature flags in place for risky changes (if applicable)

### Documentation
- [ ] README updated (if the change affects setup, usage, or architecture)
- [ ] API docs updated (if endpoints changed)
- [ ] Inline comments added where the logic isn't self-evident

## How to adapt

This is a starting point. Remove items that don't apply to your project. Add project-specific items as they emerge from PR review feedback. The checklist should reflect what actually gets caught in reviews, not an aspirational ideal.
