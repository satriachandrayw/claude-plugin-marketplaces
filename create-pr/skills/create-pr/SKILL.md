---
name: create-pr
description: Use when completing work and ready to create PR, after tests pass, or when user says "create PR", "make a PR", "/pr", or "/create-pr". Auto-commits unstaged changes, gathers git context, links issues, and generates structured PR.
version: 1.0.0
---

# Create PR Skill

Post-task PR creation with smart context gathering, auto-commit, and issue linking.

## Core Principle

Streamline PR creation after completing work by automatically committing changes, analyzing git history, linking related issues, and generating a well-structured PR with deployment checklist.

## Trigger

Activate this skill when:
- User says "create PR", "make a PR", "open a PR", "create a pull request"
- User uses `/pr` or `/create-pr` command
- Tests just passed (all green)
- Tests were just created and pass
- User indicates work is complete and ready for PR

---

## Pre-Flight: Auto-Commit Unstaged Changes

Before creating PR, check for and commit any unstaged changes.

### Check for Unstaged Changes

```bash
git status --porcelain
```

If output is not empty, proceed with auto-commit.

### Detect Commit Type from Context

| Context | Type |
|---------|------|
| New test files (*.test.*, *.spec.*) | `test:` |
| Modified test files | `test:` |
| New feature files (src/) | `feat:` |
| Bug fix keywords (fix, correct, resolve, patch) | `fix:` |
| Refactoring (no behavior change) | `refactor:` |
| Documentation (*.md, docs/) | `docs:` |
| Config/build (*.json, *.yaml, Dockerfile) | `chore:` |
| Default (unclear) | `feat:` |

### Generate Commit Message

**Format:**
```
<type>: <short description>

[optional bullet points for multiple files]

Fixes #<issue> (if issue number detected in branch/commits)
```

**Example:**
```
feat: add user authentication flow

- Add login form component
- Add auth service
- Add session management

Fixes #123
```

### Stage and Commit

```bash
# Stage all changes
git add .

# Commit with generated message
git commit -m "<type>: <description>

<body if needed>

Fixes #<issue>"
```

---

## Context Gathering

### Get Current Branch

```bash
git branch --show-current
```

### Get Main Branch

```bash
git remote show origin | grep "HEAD branch" | sed 's/.*: //'
# Or default to 'main' or 'master'
```

### Get Commits Ahead of Main

```bash
git log main..HEAD --oneline
# Or: git log origin/main..HEAD --oneline
```

### Get Diff Summary

```bash
git diff main...HEAD --stat
```

### Get Detailed Changes

```bash
git diff main...HEAD
```

### Analyze for Summary

Extract:
- What files changed
- What functionality was added/modified
- Key patterns (new feature, bug fix, refactor)

---

## Issue Detection

Extract issue numbers from branch names and commits.

### From Branch Name

Patterns to match:
- `feat/123-description` → Issue #123
- `fix/456-description` → Issue #456
- `feature/789-description` → Issue #789
- `bugfix/101-description` → Issue #101

**Command:**
```bash
git branch --show-current | grep -oE '[0-9]+' | head -1
```

### From Commit Messages

Patterns to match:
- `Fixes #123`
- `Closes #456`
- `#789: description`
- `issue #101`

**Command:**
```bash
git log main..HEAD --format="%s %b" | grep -oE '#[0-9]+' | sort -u
```

### Fetch Issue Details

For each detected issue:

```bash
gh issue view 123 --json number,title,body,labels
```

Extract:
- Title: For PR context
- Body: For understanding requirements
- Labels: For PR categorization

---

## PR Template

### Generate PR Title

**From branch prefix:**
- `feat/*` → Feature title
- `fix/*` → Fix title
- `refactor/*` → Refactor title

**From issue (if linked):**
```
[<type>] <issue-title>
```

**Examples:**
- `[FEAT] Add user authentication flow`
- `[FIX] Prevent SQL injection in login`
- `[REFACTOR] Simplify session management`

### PR Body Structure

```markdown
## Summary

<1-2 sentence description from git log analysis>

## Changes

- <change 1 from diff analysis>
- <change 2 from diff analysis>
- <change 3 from diff analysis>

## Tests

- `<test file 1>`: <what it tests>
- `<test file 2>`: <what it tests>

<If linked issues>
Fixes #<issue-number>
</If>

## Test Plan

- [ ] Run tests: `<test command detected or npm test>`
- [ ] Verify <key behavior from changes>

## Production Deployment

- [ ] Run migrations (if new migration files detected)
- [ ] Update env vars
- [ ] Update prompts (if applicable)
```

### Migration Detection

Check for migration files:

```bash
git diff main...HEAD --name-only | grep -i migration
```

If matches found, include migration step in deployment checklist.

---

## Create PR

### Push Branch (if needed)

```bash
# Check if branch tracks remote
git rev-parse --abbrev-ref --symbolic-full-name @{u}

# If no upstream, push with -u
git push -u origin <branch-name>
```

### Create PR via gh CLI

```bash
gh pr create \
  --title "<PR title>" \
  --body "$(cat <<'EOF'
<PR body content>
EOF
)"
```

### Alternative: Open PR Page

If user prefers to review before creating:

```bash
gh pr create --web
```

---

## Error Handling

| Scenario | Action |
|----------|--------|
| No commits ahead of main | Warn: "No changes to create PR. Commit your changes first." |
| Branch doesn't track remote | Push with `-u origin <branch>` |
| Issue fetch fails | Continue without issue details, warn in output |
| PR already exists | Open existing PR: `gh pr view <number> --web` |
| `gh` not authenticated | Guide user: `gh auth login` |
| No write access to repo | Suggest fork workflow |

### Check for Existing PR

```bash
gh pr list --head <branch-name> --json number,url
```

If PR exists, show URL instead of creating duplicate.

---

## Complete Workflow

```dot
digraph create_pr {
    rankdir=TB;

    "Start" [shape=doublecircle];
    "Check unstaged changes" [shape=box];
    "Unstaged changes?" [shape=diamond];
    "Detect commit type" [shape=box];
    "Stage & commit" [shape=box];
    "Get branch info" [shape=box];
    "Get git log & diff" [shape=box];
    "Extract issue numbers" [shape=box];
    "Issues found?" [shape=diamond];
    "Fetch GitHub issues" [shape=box];
    "Generate PR content" [shape=box];
    "PR exists?" [shape=diamond];
    "Open existing PR" [shape=box];
    "Push & create PR" [shape=box];
    "Done" [shape=doublecircle];

    "Start" -> "Check unstaged changes";
    "Check unstaged changes" -> "Unstaged changes?";
    "Unstaged changes?" -> "Detect commit type" [label="yes"];
    "Unstaged changes?" -> "Get branch info" [label="no"];
    "Detect commit type" -> "Stage & commit";
    "Stage & commit" -> "Get branch info";
    "Get branch info" -> "Get git log & diff";
    "Get git log & diff" -> "Extract issue numbers";
    "Extract issue numbers" -> "Issues found?";
    "Issues found?" -> "Fetch GitHub issues" [label="yes"];
    "Issues found?" -> "Generate PR content" [label="no"];
    "Fetch GitHub issues" -> "Generate PR content";
    "Generate PR content" -> "PR exists?";
    "PR exists?" -> "Open existing PR" [label="yes"];
    "PR exists?" -> "Push & create PR" [label="no"];
    "Open existing PR" -> "Done";
    "Push & create PR" -> "Done";
}
```

---

## Example Interaction

**User:** `/pr`

**Create-PR:**
```
🚀 Creating PR...

📝 Checking for unstaged changes...
   Found 2 unstaged files
   Detected type: feat (new feature files)
   Committing: "feat: add user profile component"
   ✓ Committed

📊 Gathering context...
   Branch: feat/142-add-user-profile
   Commits: 3 ahead of main
   Files changed: 5

🔗 Detecting linked issues...
   Found #142 in branch name
   Fetching issue details... ✓

📝 Generating PR content...

🚀 Creating PR...
   ✓ PR #15 created: https://github.com/owner/repo/pull/15

✅ Done!
   - Auto-committed 2 files
   - Linked issue #142
   - PR created with deployment checklist
```

---

## Notes

- **Conventional commits** - Auto-detect type from changed files
- **Smart issue linking** - Extract from branch name and commit messages
- **Migration detection** - Auto-add migration step if migration files changed
- **Graceful failures** - Continue without issues if fetch fails
- **No duplicates** - Check for existing PR before creating

---
