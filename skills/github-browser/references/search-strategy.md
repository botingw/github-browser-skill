# Search Strategy Reference

When researching a topic or ecosystem on GitHub, **don't just use keyword repo search**. Think first, then search through multiple channels.

## Step 1: Parse the Intent

Before searching, ask: what does "related to X" actually mean?

| User says | Likely means | Best search channel |
|-----------|-------------|-------------------|
| "X 相關的 repos" / "repos related to X" | Broad — all of the below | Multi-channel |
| "用了 X 的專案" / "projects using X" | Code imports or depends on X | Code search + dependency search |
| "X 生態圈" / "X ecosystem" | Official + community projects | Org browsing + topic search |
| "X 的替代品" / "alternatives to X" | Similar purpose, different impl | Repo search + awesome lists |
| "X 的教學" / "X tutorials" | Learning resources | Repo search with "tutorial"/"example"/"course" |

## Step 2: Multi-Channel Search

Use at least 2-3 channels. Each covers a different dimension of "related".

### Channel A: Repo Search (name/description)

Finds repos that **mention X** in their name or description.

```bash
gh search repos "langchain" --sort stars --limit 20 \
  --json fullName,description,stargazersCount,language
```

**Limitation:** Misses repos that use X but have their own brand name (e.g. "DeerFlow" uses LangGraph but the name doesn't say so).

### Channel B: Code Search (actual usage)

Finds repos that **actually import or use X** in their code. This catches what Channel A misses.

```bash
# Python imports
gh search code "from langchain" --language python --sort indexed
gh search code "import langgraph" --language python --sort indexed

# TypeScript/JavaScript imports
gh search code "require('langchain')" --language typescript
gh search code "from 'langchain'" --language typescript
```

**Gotcha:** Code search returns individual file matches, not repos. Extract unique repo names from results:

```bash
gh search code "from langgraph" --language python \
  --json repository --jq '[.[].repository.fullName] | unique[]'
```

### Channel C: Dependency Search

Finds repos that **declare X as a dependency** in their config files.

```bash
# Python: requirements.txt
gh search code "langchain" --filename requirements.txt --sort indexed

# Python: pyproject.toml
gh search code "langchain" --filename pyproject.toml --sort indexed

# Node: package.json
gh search code "langchain" --filename package.json --sort indexed
```

### Channel D: Topic Search

Finds repos that are **tagged with X** as a GitHub topic.

```bash
gh search repos --topic langchain --sort stars --limit 20
gh search repos --topic ai-agent --sort stars --limit 20
gh search repos --topic langgraph --sort stars --limit 20
```

### Channel E: Org Browsing

Finds repos **published by X's official organization**.

```bash
# List all repos from an org, sorted by stars
gh api "orgs/langchain-ai/repos?sort=stars&per_page=30&direction=desc" \
  --jq '.[] | "⭐ \(.stargazers_count) | \(.full_name) | \(.description)"'
```

**Note:** `gh api` returns `stargazers_count` (underscore). `gh search repos --json` returns `stargazersCount` (camelCase). `gh repo view --json` returns `stargazerCount` (camelCase, no 's'). These are all different.

### Channel F: Awesome Lists

Many ecosystems have "awesome-X" repos curated by the community.

```bash
gh search repos "awesome langchain" --sort stars --limit 5
gh search repos "awesome ai-agents" --sort stars --limit 5
```

Then read the list:

```bash
gh api repos/{owner}/awesome-langchain/contents/README.md --jq '.content' | base64 -d
```

## Step 3: Cross-Reference and Rank

After collecting results from multiple channels:

1. **Deduplicate** — the same repo may appear in multiple channels
2. **Prioritize** — repos appearing in 2+ channels are more likely to be relevant
3. **Sort by stars** as a rough popularity proxy, but don't ignore low-star repos that appear in multiple channels

## Step 4: Deep Dive Top Candidates

For the top repos you want to present:

```bash
# Quick description (clean, includes metadata)
gh repo view {owner}/{repo}

# Or for batch processing (parallel-friendly but raw markdown):
gh api repos/{owner}/{repo}/contents/README.md --jq '.content' | base64 -d | head -50
```

Tip: `gh repo view` gives cleaner output but can't run in parallel. Use `gh api` when you need to read 5+ repos at once.

## Step 5: Iterate

From your first round of results, look for:
- **Recurring org names** — browse that org for more repos
- **New topics** you didn't search for — search those topics
- **"See also" / "Related projects"** sections in READMEs — follow those leads
- **"Used by" signals** — if a repo lists companies/projects using it, those are research leads

## Common Gotchas

### No boolean OR in search

`gh search repos "langchain OR langgraph"` does NOT do boolean OR. It searches for the literal string "langchain OR langgraph".

**Correct approach:** Run separate searches and merge results.

```bash
# WRONG
gh search repos "langchain OR langgraph" --sort stars

# RIGHT — two separate searches
gh search repos "langchain" --sort stars --limit 15
gh search repos "langgraph" --sort stars --limit 15
```

### Field name inconsistencies across APIs

| Context | Star count field name |
|---------|----------------------|
| `gh search repos --json` | `stargazersCount` |
| `gh repo view --json` | `stargazerCount` |
| `gh api repos/...` (REST API) | `stargazers_count` |

Always check the correct field name for the API you're using.

### Code search rate limit

Code search is limited to **10 requests per minute**. If doing multi-channel research, space out code search calls or batch them.

### Code search only indexes default branch

Repos with no activity in the last year, forked repos (unless more stars than parent), and files over 384KB are not indexed.
