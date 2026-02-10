# Issues & PRs Reference

## Issues

### List issues with filters

```bash
# Open issues (default)
gh issue list -R {owner}/{repo}

# Closed issues
gh issue list -R {owner}/{repo} --state closed

# Filter by label
gh issue list -R {owner}/{repo} --label "bug"

# Filter by assignee
gh issue list -R {owner}/{repo} --assignee username

# Filter by milestone
gh issue list -R {owner}/{repo} --milestone "v1.0"

# Show more results (default is 30)
gh issue list -R {owner}/{repo} --limit 100

# JSON output with specific fields
gh issue list -R {owner}/{repo} --json number,title,labels,assignees,createdAt
```

### Read issue content

```bash
# View issue body
gh issue view {n} -R {owner}/{repo}

# View with all comments
gh issue view {n} -R {owner}/{repo} --comments

# JSON output
gh issue view {n} -R {owner}/{repo} --json title,body,comments,labels,assignees,state
```

## Pull Requests

### List PRs with filters

```bash
# Open PRs (default)
gh pr list -R {owner}/{repo}

# Merged PRs
gh pr list -R {owner}/{repo} --state merged

# Closed (not merged) PRs
gh pr list -R {owner}/{repo} --state closed

# Filter by base branch
gh pr list -R {owner}/{repo} --base main

# Filter by author
gh pr list -R {owner}/{repo} --author username

# Filter by label
gh pr list -R {owner}/{repo} --label "needs-review"

# JSON output
gh pr list -R {owner}/{repo} --json number,title,author,baseRefName,headRefName,state
```

### Read PR content

```bash
# View PR description
gh pr view {n} -R {owner}/{repo}

# View with comments
gh pr view {n} -R {owner}/{repo} --comments

# JSON output with review info
gh pr view {n} -R {owner}/{repo} --json title,body,comments,reviews,state,mergeable,statusCheckRollup
```

### PR Diff

```bash
# Full diff
gh pr diff {n} -R {owner}/{repo}

# Diff stat (summary of changes)
gh pr diff {n} -R {owner}/{repo} --stat

# List changed files with patch via API (structured)
gh api repos/{owner}/{repo}/pulls/{n}/files \
  --jq '.[] | "\(.status)\t\(.changes)\t\(.filename)"'

# Get the actual patch for a specific file in a PR
gh api repos/{owner}/{repo}/pulls/{n}/files \
  --jq '.[] | select(.filename == "{path}") | .patch'
```

### PR Review Comments (inline comments on code)

```bash
# List all review comments
gh api repos/{owner}/{repo}/pulls/{n}/comments \
  --jq '.[] | "---\n\(.user.login) on \(.path):\(.line // .original_line):\n\(.body)\n"'

# List reviews (approve/request changes/comment)
gh api repos/{owner}/{repo}/pulls/{n}/reviews \
  --jq '.[] | "\(.user.login): \(.state) - \(.body)"'
```

### CI / Checks Status

```bash
# Check status summary
gh pr checks {n} -R {owner}/{repo}

# JSON output for programmatic use
gh pr view {n} -R {owner}/{repo} --json statusCheckRollup \
  --jq '.statusCheckRollup[] | "\(.name): \(.conclusion // .status)"'
```

## Issue & PR Timeline

For detailed event history (label changes, assignments, references, etc.):

```bash
gh api repos/{owner}/{repo}/issues/{n}/timeline \
  --jq '.[] | "\(.event) by \(.actor.login // "system") at \(.created_at)"'
```

Note: Both issues and PRs share the same timeline API endpoint under `/issues/`.
