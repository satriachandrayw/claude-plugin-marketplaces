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
