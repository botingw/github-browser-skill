# GitHub Browser Skill

A Claude Code skill that teaches AI agents to efficiently browse and explore GitHub repositories using the `gh` CLI. This skill provides optimized command patterns for reading repository content, issues, PRs, and performing searches—all without leaving the terminal.

## Features

- **Read-only GitHub operations** via `gh` CLI and GitHub API
- **Smart URL parsing** — paste any GitHub URL and get the right content
- **Efficient browsing patterns** — optimized commands for common tasks
- **Multi-channel search strategy** — systematic approach to discovering repos and code
- **Progressive disclosure design** — core commands in main file, advanced scenarios in references

## What This Skill Teaches

The skill provides AI agents with:

1. **URL-to-action mapping** — automatically determine what to do with GitHub URLs
2. **Decision trees** — navigate from intent ("I want to understand a repo") to the right commands
3. **Command patterns** — tested `gh` CLI commands with `jq` filters for precise data extraction
4. **Search strategies** — multi-channel approach for discovering related repositories and code
5. **Advanced scenarios** — handling large files, PR review comments, complex filters

## Typical Use Cases

- Browse repository structure and read files
- View issues and pull requests with full context
- Explore PR diffs and review comments
- Search code, issues, and repositories
- Research a technology ecosystem (e.g., "find popular React UI libraries")
- Investigate a specific GitHub URL shared by someone

## Installation

### For Claude Code

1. Copy the skill to your Claude plugins directory:
   ```bash
   cp -r skills/github-browser ~/.claude/skills/
   ```

2. Verify installation:
   ```bash
   ls ~/.claude/skills/github-browser
   ```

### For Other AI Agents

Package the skill into a distributable `.skill` file:

```bash
# If you have the skill-creator tools
python scripts/package_skill.py skills/github-browser

# Or manually zip the directory
cd skills && zip -r github-browser.skill github-browser/
```

## Prerequisites

The skill requires the [GitHub CLI (`gh`)](https://cli.github.com/) to be installed and authenticated:

```bash
# Install gh CLI (macOS)
brew install gh

# Authenticate
gh auth login

# Verify
gh auth status
```

## Structure

```
skills/github-browser/
├── SKILL.md                    # Main entry point with triggers and core commands
├── DESIGN.md                   # Design rationale and iteration guidelines
└── references/
    ├── repo-exploration.md     # File trees, reading files, metadata, branches
    ├── issues-and-prs.md       # Issues, PRs, diffs, review comments
    ├── search.md               # Code/issue/repo search with advanced filters
    └── search-strategy.md      # Multi-channel search methodology
```

## How It Works

1. **Skill triggering** — When you give Claude Code a GitHub URL or ask to explore a repository, this skill automatically activates
2. **Progressive loading** — The main SKILL.md loads first with core commands; detailed references load on-demand
3. **Command templates** — Instead of generating scripts, the skill teaches command patterns that AI agents execute directly
4. **Best practices** — All commands have been tested and optimized for real-world usage

## Example Interactions

**Understanding a repository:**
```
User: "Check out https://github.com/facebook/react"
→ AI uses: gh repo view facebook/react
→ AI uses: gh api repos/facebook/react/git/trees/main?recursive=1
```

**Reading a specific file:**
```
User: "Show me the package.json from the React repo"
→ AI uses: gh api repos/facebook/react/contents/package.json --jq '.content' | base64 -d
```

**Exploring a PR:**
```
User: "What changed in PR #1234 of react?"
→ AI uses: gh pr view 1234 -R facebook/react --comments
→ AI uses: gh pr diff 1234 -R facebook/react
```

**Researching an ecosystem:**
```
User: "Find popular TypeScript testing libraries"
→ AI uses multi-channel search:
  - gh search repos "typescript testing" --language typescript --sort stars
  - gh search code "import.*test" --language typescript
  - gh search repos --topic testing --topic typescript --sort stars
```

## Design Principles

1. **No custom scripts** — Teach command patterns using native `gh` CLI + `jq`, not bash scripts
2. **Concise main file** — SKILL.md stays under 150 lines; details go to references/
3. **Tested commands** — All command patterns verified with real `gh` CLI
4. **Extensible architecture** — Structure allows future addition of write operations (create issue, comment, merge)

## Limitations

- **Read-only** — Current version does not include write operations (creating issues, commenting, merging PRs)
- **Requires authentication** — Must have `gh auth login` completed
- **Rate limits** — GitHub API rate limits apply (especially for code search: 10 requests/min)
- **Large repositories** — Very large repos may have truncated file trees (skill includes fallback strategies)

## Future Enhancements

The skill architecture is designed to support future additions:

- Write operations (create issues, comment on PRs, merge)
- GitHub Actions workflows viewing
- GitHub Discussions browsing
- Organization and team exploration
- Advanced webhook and integration management

## Contributing

This skill follows the [Skill Creator guidelines](https://docs.anthropic.com/claude/docs/claude-code-skills) for Claude Code skills.

When contributing:

1. **Test all commands** — Verify with actual `gh` CLI before adding
2. **Keep SKILL.md lean** — Core commands only; move detailed scenarios to references/
3. **Follow progressive disclosure** — Don't load unnecessary context
4. **Document in DESIGN.md** — Record architectural decisions for future maintainers

## License

[Include your license here, e.g., MIT, Apache 2.0]

## Author

Created by [Your Name] for the Claude Code community.

## Acknowledgments

- Built using the [Claude Code Skill Creator](https://github.com/anthropics/claude-code)
- Powered by [GitHub CLI](https://cli.github.com/)
