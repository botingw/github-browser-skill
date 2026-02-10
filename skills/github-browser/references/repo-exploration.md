# Repo Exploration Reference

## File Tree

### Full recursive tree

```bash
gh api repos/{owner}/{repo}/git/trees/{branch}?recursive=1 \
  --jq '.tree[] | select(.type == "blob") | .path'
```

### Tree with file sizes

```bash
gh api repos/{owner}/{repo}/git/trees/{branch}?recursive=1 \
  --jq '.tree[] | select(.type == "blob") | "\(.size)\t\(.path)"'
```

### Only show specific directory subtree

Filter with `select(.path | startswith("src/"))`.

### Tree is truncated?

If the API response has `"truncated": true`, the repo is very large. Fall back to listing directories level by level:

```bash
gh api repos/{owner}/{repo}/contents/{path} --jq '.[].name'
```

## Reading Files

### Standard file (≤1MB)

```bash
gh api repos/{owner}/{repo}/contents/{path} --jq '.content' | base64 -d
```

### File from a specific branch/tag

```bash
gh api repos/{owner}/{repo}/contents/{path}?ref={branch_or_tag} --jq '.content' | base64 -d
```

### Large files (>1MB)

The contents API returns `"errors"` for files >1MB. Use the blob API instead:

1. Get the file's SHA from the tree:
```bash
gh api repos/{owner}/{repo}/git/trees/{branch}?recursive=1 \
  --jq '.tree[] | select(.path == "{path}") | .sha'
```

2. Fetch the blob:
```bash
gh api repos/{owner}/{repo}/git/blobs/{sha} --jq '.content' | base64 -d
```

### Binary files

Do not attempt to base64-decode binary files (images, compiled artifacts). Instead, report the file's existence and size from the tree API.

## Directory Contents

### List a directory

```bash
gh api repos/{owner}/{repo}/contents/{path} --jq '.[] | "\(.type)\t\(.name)"'
```

### List with sizes

```bash
gh api repos/{owner}/{repo}/contents/{path} --jq '.[] | "\(.type)\t\(.size)\t\(.name)"'
```

## Repo Metadata

### Overview (README + description)

```bash
gh repo view {owner}/{repo}
```

### Structured metadata (JSON)

```bash
gh repo view {owner}/{repo} --json name,description,url,stargazerCount,forkCount,primaryLanguage,licenseInfo,topics,defaultBranchRef
```

### Languages breakdown

```bash
gh api repos/{owner}/{repo}/languages
```

## Branches & Tags

### List branches

```bash
gh api repos/{owner}/{repo}/branches --jq '.[].name'
```

### List tags

```bash
gh api repos/{owner}/{repo}/tags --jq '.[].name'
```

### Default branch name

```bash
gh repo view {owner}/{repo} --json defaultBranchRef --jq '.defaultBranchRef.name'
```

## Releases

### List releases

```bash
gh release list -R {owner}/{repo}
```

### View latest release

```bash
gh release view -R {owner}/{repo}
```

### View specific release

```bash
gh release view {tag} -R {owner}/{repo}
```

## Commits

### Recent commits

```bash
gh api repos/{owner}/{repo}/commits --jq '.[] | "\(.sha[:7]) \(.commit.message | split("\n")[0])"' | head -20
```

### Commits on a specific path

```bash
gh api "repos/{owner}/{repo}/commits?path={path}" --jq '.[] | "\(.sha[:7]) \(.commit.message | split("\n")[0])"'
```
