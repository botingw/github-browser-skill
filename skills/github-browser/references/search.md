# Search Reference

## Code Search

### Basic code search

```bash
gh search code "query" -R {owner}/{repo}
```

### Filter by language

```bash
gh search code "query" -R {owner}/{repo} --language python
```

### Filter by file path

```bash
gh search code "query" -R {owner}/{repo} --filename "*.go"
```

### JSON output

```bash
gh search code "query" -R {owner}/{repo} --json repository,path,textMatches
```

### Search across all repos of an org/user

```bash
gh search code "query" --owner {owner}
```

### Exact string match

Wrap in quotes inside the query:

```bash
gh search code '"exact phrase"' -R {owner}/{repo}
```

## Issue & PR Search

### Basic search

```bash
gh search issues "query" -R {owner}/{repo}
```

### Filter by type (issue vs PR)

```bash
# Issues only
gh search issues "query" -R {owner}/{repo} --type issue

# PRs only
gh search issues "query" -R {owner}/{repo} --type pr
```

### Filter by state

```bash
gh search issues "query" -R {owner}/{repo} --state open
gh search issues "query" -R {owner}/{repo} --state closed
```

### Sort results

```bash
# By most recently updated
gh search issues "query" -R {owner}/{repo} --sort updated --order desc

# By most comments
gh search issues "query" -R {owner}/{repo} --sort comments --order desc
```

### JSON output

```bash
gh search issues "query" -R {owner}/{repo} --json number,title,state,url,updatedAt
```

## Repo Search

### Basic search

```bash
gh search repos "query"
```

### Filter by language

```bash
gh search repos "query" --language rust
```

### Filter by stars

```bash
gh search repos "query" --stars ">1000"
```

### Sort by stars

```bash
gh search repos "query" --sort stars --order desc
```

### JSON output

```bash
gh search repos "query" --json fullName,description,stargazersCount,language,url
```

## Search Tips

### No Boolean OR

GitHub search does NOT support `OR` / `AND` boolean operators. `gh search repos "langchain OR langgraph"` searches for the literal string. **Run separate searches instead.**

### Field Name Inconsistencies

The star count field has different names depending on the API:

| Context | Field name |
|---------|------------|
| `gh search repos --json` | `stargazersCount` |
| `gh repo view --json` | `stargazerCount` (no 's') |
| `gh api repos/...` (REST) | `stargazers_count` (underscore) |

### Rate Limits

- Code search: 10 requests per minute (authenticated)
- Issue/repo search: 30 requests per minute (authenticated)
- If you hit rate limits, wait before retrying. Do not retry in a tight loop.

### Query Syntax

GitHub code search supports qualifiers directly in the query string:

```bash
# Search by file extension
gh search code "func main" -R {owner}/{repo} --language go

# Search in specific path
gh search code "TODO" -R {owner}/{repo} --filename "src/*"
```

### Result Limits

- Default: 30 results
- Maximum per request: `--limit 100`
- For more results, refine your query rather than paginating

### When Code Search Doesn't Work

GitHub code search has limitations:
- Only indexes the default branch
- Doesn't index forked repos (unless they have more stars than parent)
- Files over 384KB are not indexed
- Repos must have had activity in the last year

If code search fails, fall back to cloning the repo and searching locally with `grep`/`rg`.
