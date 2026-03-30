# GitHub Integration (Source Control)

## What it does

PR management, code review, workflow monitoring, secrets management, branch protection. Accessed primarily via the `gh` CLI.

## Auth

Authenticated via `gh` CLI (already logged in). For CI workflows that need to trigger other workflows, use a Personal Access Token (PAT).

```
Token location: .env → AUTO_VERSION_PAT (or similar)
```

## Common Operations

### PRs

```bash
# List open PRs across the org
gh search prs --owner [ORG] --state open --limit 20

# List open PRs by author
gh search prs --owner [ORG] --state open --author [USERNAME]

# View a specific PR
gh pr view [NUMBER] --repo [ORG]/[REPO]

# View PR diff
gh pr diff [NUMBER] --repo [ORG]/[REPO]

# Create a PR
gh pr create --repo [ORG]/[REPO] --title "title" --body "body"

# View PR comments
gh api repos/[ORG]/[REPO]/pulls/[NUMBER]/comments
```

### Workflows

```bash
# List recent workflow runs
gh run list --repo [ORG]/[REPO] --limit 5

# View a specific run
gh run view [RUN_ID] --repo [ORG]/[REPO]

# Trigger a workflow dispatch
gh workflow run [WORKFLOW_FILE] --repo [ORG]/[REPO] --ref main
```

### Repository settings

```bash
# Set merge options (squash only, auto-delete branches)
gh api repos/[ORG]/[REPO] -X PATCH \
  -f allow_squash_merge=true \
  -f allow_merge_commit=false \
  -f allow_rebase_merge=false \
  -f delete_branch_on_merge=true

# Set branch protection
gh api repos/[ORG]/[REPO]/branches/main/protection -X PUT \
  -H "Accept: application/vnd.github+json" \
  --input - <<'EOF'
{
  "required_pull_request_reviews": {"required_approving_review_count": 0},
  "enforce_admins": true,
  "required_status_checks": null,
  "restrictions": null
}
EOF

# Manage secrets
gh secret set SECRET_NAME --repo [ORG]/[REPO] --body "value"
gh secret list --repo [ORG]/[REPO]
```

## Gotchas

- `GITHUB_TOKEN` (default in Actions) cannot trigger other workflows. Use a PAT for cross-workflow triggers.
- `gh search prs` searches across the org; `gh pr list` is scoped to a single repo.
- Branch protection with `enforce_admins: true` blocks everyone including admins — add bypass allowances for automation users.
