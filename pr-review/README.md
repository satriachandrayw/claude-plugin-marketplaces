# PR-Review Plugin

A comprehensive PR code review plugin for Claude Code that orchestrates multiple specialized analyzers to provide thorough, structured code reviews with impact analysis, security scanning, convention checking, and architectural assessment.

## Features

- **Impact Analysis** - Maps the blast radius of changes including direct callers, transitive dependencies, and related modules
- **Security Scanning** - Detects injection risks, authentication issues, and data exposure vulnerabilities
- **Convention Checking** - Validates naming conventions, structure limits, TypeScript patterns, and import organization
- **Architecture Review** - Identifies pattern violations, coupling increases, and layer violations
- **PR Classification** - Auto-classifies PRs as feature/bug-fix/chore/refactor
- **Dual Output** - Generates local markdown reports or CI PR comments via GitHub API
- **Fallback Support** - Works without auggie using built-in grep/glob tools

## Requirements

### Required

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code)
- Git

### Optional (for enhanced context)

- [auggie CLI](https://www.augmentcode.com/) with `codebase-retrieval` MCP tool
- Augment account for auggie authentication

## Installation

In Claude Code, run:

```bash
/plugin add https://github.com/satriachandrayw/skills
```

The plugin will be available after installation.

## Usage

### Local (Manual)

Review the current branch against main:

```bash
/pr-review
```

Review a specific commit range:

```bash
/pr-review HEAD~5 HEAD
```

Review specific SHAs:

```bash
/pr-review abc123 def456
```

Review a specific PR by number:

```bash
/pr-review --pr 142
```

### CI (GitHub Actions)

The plugin automatically detects CI environments and adapts its behavior:

| Variable | Description |
|----------|-------------|
| `CI=true` | Indicates CI environment |
| `GITHUB_ACTIONS=true` | GitHub Actions detected |
| `GITHUB_TOKEN` | Required for posting PR comments |
| `PR_NUMBER` | PR number to review |

#### Example GitHub Actions Workflow

```yaml
name: PR Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for accurate diff

      - name: Setup Claude Code
        run: |
          # Install Claude Code CLI
          npm install -g @anthropic-ai/claude-code

      - name: Run PR Review
        env:
          CI: true
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          PR_NUMBER: ${{ github.event.pull_request.number }}
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          claude /pr-review
```

## How It Works

The plugin operates in three phases:

### Phase 1: Gather Context

Collects information about the changes:
- Git diff between base and head commits
- Commit messages in the range
- Changed files with statistics
- Impact trace (callers, dependencies, related modules)

### Phase 2: Analyze (Parallel)

Runs specialized analyzers concurrently:
- **Security Analyzer** - Detects vulnerabilities (SQL injection, XSS, hardcoded credentials, auth issues)
- **Convention Checker** - Checks naming, structure limits, TypeScript patterns, imports
- **Impact Analyzer** - Assesses blast radius and architectural implications
- **PR Classifier** - Determines PR type and confidence

### Phase 3: Synthesize

Combines findings into a unified report with:
- Classification summary
- Impact assessment
- Security and convention findings
- Architectural notes
- Actionable recommendations

## Output Formats

### Local Report

When running locally, generates a markdown report with:
- Summary table (type, confidence, SHAs)
- Changed files list
- Impact analysis tables
- Architectural notes
- Security and convention findings
- Verdict with recommendations

### CI Comment

When running in CI, posts a GitHub review with:
- Review event type (APPROVE, REQUEST_CHANGES, COMMENT)
- Summary body with findings counts
- Line-level comments on specific issues

## Configuration

The plugin works out of the box with sensible defaults. No additional configuration is required.

### Environment Variables

| Variable | Purpose | Required |
|----------|---------|----------|
| `CI` | Enable CI mode | No (auto-detected) |
| `GITHUB_TOKEN` | GitHub API access | Yes (CI mode) |
| `PR_NUMBER` | PR to review | Yes (CI mode) |

## Error Handling

| Scenario | Behavior |
|----------|----------|
| Invalid SHAs | Error with message "Could not resolve SHA" |
| No changes | Warning: "No changes detected" |
| Analyzer timeout | Continue with partial results |
| Template not found | Use inline fallback template |
| `gh` CLI unavailable (CI) | Error: "GitHub CLI required for CI mode" |
| `codebase-retrieval` unavailable | Fall back to Grep/Glob tools |

## Project Structure

```
pr-review/
  .claude-plugin/
    plugin.json              # Plugin manifest
  skills/
    pr-review/
      SKILL.md               # Main orchestrator skill
  agents/
    gather-context.md        # Context gathering agent
    security-analyzer.md     # Security vulnerability detection
    convention-checker.md    # Code convention validation
    impact-analyzer.md       # Impact and architecture analysis
    pr-classifier.md         # PR type classification
  templates/
    report-local.md          # Local markdown report template
    report-ci-comment.md     # CI PR comment template
```

## Troubleshooting

### "Could not resolve SHA"

Ensure you're in a git repository and the commit exists:
```bash
git rev-parse <sha>
```

### "GitHub CLI required for CI mode"

Install the GitHub CLI:
```bash
brew install gh  # macOS
# or
sudo apt install gh  # Ubuntu
```

### "GITHUB_TOKEN required for CI mode"

Add the token to your workflow:
```yaml
env:
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Limited context without auggie

For enhanced context analysis, install auggie CLI and authenticate:
```bash
npm install -g @augmentcode/auggie
auggie auth login
```

## Author

Satria Chandra (satriachandrayw@gmail.com)

## License

MIT
