---
name: github-browser
description: >
  Use this skill when you need to browse, explore, or read content from GitHub repositories.
  Triggers: reading repo files, viewing issues/PRs, searching code on GitHub,
  exploring repo structure, reading PR diffs/reviews, or when given a GitHub URL to investigate.
  This skill teaches you the optimal gh CLI commands for each scenario.
argument-hint: "[github-url-or-repo]"
---

# GitHub Browser Skill

Browse and read GitHub content efficiently using `gh` CLI. **Read-only operations only.**

## Prerequisites

Before running any command, verify authentication:

```bash
gh auth status
```

If not authenticated, tell the user to run `gh auth login`.

## URL Parsing

When given a GitHub URL, parse it to determine the action:

| URL Pattern | Type | Action |
|---|---|---|
| `github.com/{owner}/{repo}` | Repo root | View repo overview |
| `github.com/{owner}/{repo}/tree/{branch}/{path}` | Directory | List directory contents |
| `github.com/{owner}/{repo}/blob/{branch}/{path}` | File | Read file content |
| `github.com/{owner}/{repo}/issues/{n}` | Issue | View issue details |
| `github.com/{owner}/{repo}/pull/{n}` | PR | View PR details |
| `github.com/{owner}/{repo}/pull/{n}/files` | PR diff | View changed files |

## Decision Tree

### "I want to understand a repo"
1. Quick overview → `gh repo view {owner}/{repo}`
2. File tree → See [repo-exploration.md](references/repo-exploration.md)
3. Read a specific file → See [repo-exploration.md](references/repo-exploration.md)

### "I want to read an issue or PR"
1. List issues → `gh issue list -R {owner}/{repo}`
2. Read issue → `gh issue view {n} -R {owner}/{repo} --comments`
3. List PRs → `gh pr list -R {owner}/{repo}`
4. Read PR → `gh pr view {n} -R {owner}/{repo} --comments`
5. PR diff / review comments → See [issues-and-prs.md](references/issues-and-prs.md)

### "I want to search"
1. Search code → `gh search code "query" -R {owner}/{repo}`
2. Search issues → `gh search issues "query" -R {owner}/{repo}`
3. Search repos → `gh search repos "query"`
4. Advanced search → See [search.md](references/search.md)

### "I want to research a topic / find repos in an ecosystem"

**STOP — don't just keyword search.** First think about search strategy:

1. **Parse intent** — what does "related to X" mean? Uses X in code? Part of X org? Mentions X in description?
2. **Use multiple channels** (at least 2-3):
   - Repo search (name/description): `gh search repos "X" --sort stars`
   - Code search (actual usage): `gh search code "import X" --language python`
   - Dependency search: `gh search code "X" --filename requirements.txt`
   - Topic search: `gh search repos --topic X --sort stars`
   - Org browsing: `gh api orgs/{X-org}/repos?sort=stars&per_page=30`
3. **Cross-reference** — repos appearing in multiple channels are more relevant
4. **Iterate** — discover new orgs/topics from results, search again

→ See [search-strategy.md](references/search-strategy.md) for full playbook

## Core Command Quick Reference

### Repo

```bash
# Repo overview (README + metadata)
gh repo view {owner}/{repo}

# File tree (recursive)
gh api repos/{owner}/{repo}/git/trees/{branch}?recursive=1 --jq '.tree[] | .path'

# Read a file (≤1MB)
gh api repos/{owner}/{repo}/contents/{path} --jq '.content' | base64 -d

# Read a file from a specific branch
gh api repos/{owner}/{repo}/contents/{path}?ref={branch} --jq '.content' | base64 -d

# List directory
gh api repos/{owner}/{repo}/contents/{path} --jq '.[].name'
```

### Issues & PRs

```bash
# List open issues
gh issue list -R {owner}/{repo}

# View issue with comments
gh issue view {n} -R {owner}/{repo} --comments

# List open PRs
gh pr list -R {owner}/{repo}

# View PR with comments
gh pr view {n} -R {owner}/{repo} --comments

# PR changed files summary
gh pr diff {n} -R {owner}/{repo} --stat

# PR full diff
gh pr diff {n} -R {owner}/{repo}

# CI/checks status
gh pr checks {n} -R {owner}/{repo}
```

### Search

```bash
# Search code in a repo
gh search code "query" -R {owner}/{repo}

# Search code by language
gh search code "query" -R {owner}/{repo} --language go

# Search issues/PRs
gh search issues "query" -R {owner}/{repo}

# Search repos
gh search repos "query" --language python
```

## Detailed References

For complex scenarios, refer to:
- [Repo exploration](references/repo-exploration.md) — file trees, large files, metadata, branches, releases
- [Issues and PRs](references/issues-and-prs.md) — filters, PR diffs, review comments, timeline
- [Search](references/search.md) — advanced filters, JSON output, rate limits
- [Search strategy](references/search-strategy.md) — multi-channel search thinking framework for topic/ecosystem research
