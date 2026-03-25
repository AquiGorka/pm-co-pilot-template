# Posting Review Comments on GitHub PRs

After completing a review, post findings as inline PR comments using the GitHub Reviews API.

## Inline comments (on lines in the diff)

Use the Reviews API — it accepts `line` numbers (the single-comment API requires `position` which is a diff offset and much harder to compute).

```bash
COMMIT_ID=$(gh pr view <PR_NUMBER> --repo <OWNER>/<REPO> --json headRefOid --jq '.headRefOid')

cat <<PAYLOAD | gh api repos/<OWNER>/<REPO>/pulls/<PR_NUMBER>/reviews --method POST --input -
{
  "commit_id": "${COMMIT_ID}",
  "event": "COMMENT",
  "body": "",
  "comments": [
    {
      "path": "src/path/to/file.ts",
      "line": 42,
      "side": "RIGHT",
      "body": "Description of the issue.\n\n**Severity: HIGH**"
    }
  ]
}
PAYLOAD
```

**Key constraints:**
- `line` must be a line number that exists in the diff (new file side). You cannot comment on unchanged lines.
- `side` should be `"RIGHT"` (new file side) for added/modified lines.
- Multiple comments can be included in the `comments` array in a single API call.
- `commit_id` must be the HEAD commit of the PR — get it via `gh pr view`.
- `event` can be `"COMMENT"`, `"APPROVE"`, or `"REQUEST_CHANGES"`.

## Finding the correct line number

The `line` parameter is the line number in the **new version of the file**, not a diff offset. To find it:

1. Get the diff: `gh pr diff <PR_NUMBER> --repo <OWNER>/<REPO>`
2. Find the hunk header: `@@ -old_start,old_count +new_start,new_count @@`
3. Count only `+` (added) and ` ` (context) lines from `new_start` to reach your target line. Do not count `-` (removed) lines.

## General comments (for issues in unchanged code)

If the issue is in code that wasn't changed in the PR (not in the diff), you cannot post an inline comment. Use a general PR comment instead:

```bash
gh pr comment <PR_NUMBER> --repo <OWNER>/<REPO> --body "Description of issues in unchanged code"
```

## Workflow

1. Review the PR and collect findings.
2. For each finding, determine if the relevant line is in the diff.
3. Post inline comments for findings on diff lines (batch them in one API call).
4. Post a single general comment for findings on unchanged code.
5. If asked to post only specific findings, do exactly that — no more.
