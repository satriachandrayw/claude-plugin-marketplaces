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
