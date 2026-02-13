---
name: agent-knowledge
description: Agent Knowledge Base - Captures learnings from external AI agents (Codex, etc.) into structured knowledge base. Use when user says "capture learnings", "agent learned", "what did codex discover", or after agent work sessions.
version: 1.0.0
---

# Agent Knowledge Base Skill

You manage a knowledge base that captures learnings, discoveries, and patterns from external AI agents (Codex, others) that work via stdout/stderr.

## Trigger

Activate this skill when user:
- Says "capture learnings", "agent learned", "what did codex discover"
- Mentions "codex found something" or "agent discovered"
- Asks to document agent session outcomes
- Wants to preserve insights from agent work

## Background

From `AGENTS.md`, we know agent behavior:
- **Codex**: Makes parallel edits, continues on unexpected changes, uses skills from `.codex/skills/` and `~/.codex/skills/`
- **Agents output to stdout/stderr** (not log files)
- Therefore, we use manual trigger + git diff analysis approach

## Process Flow

### 1. Analyze Recent Changes

When triggered, run:

```bash
git log --oneline -10
git diff HEAD~5..HEAD --stat
```

Identify:
- Files changed (paths indicate topics)
- Features touched (batch-image, auth, database, api, etc.)
- Recent commits that may contain agent work

### 2. Detect Topic

Based on paths changed, map to topics:

| Paths | Topic | KB File |
|--------|--------|----------|
| `apps/pages/src/routes/batch-*`, `batch-image` | Batch Processing | `batch-processing.md` |
| `apps/worker/src/routes/*`, `apps/worker/src/handlers/*` | API Contracts | `api-contracts.md` |
| `apps/worker/src/storage/`, `d1-` | Database Patterns | `database-patterns.md` |
| `apps/pages/src/lib/session`, `auth`, `Auth` | Auth Sessions | `auth-sessions.md` |
| `apps/content-generator`, `image`, `photo` | Image Generation | `image-generation.md` |
| `apps/pages/src/routes/admin/*` | Admin Panel | `admin-panel.md` |
| `apps/pages/src/components/*` | UI Components | `ui-components.md` |
| `functions-migration`, `migrate` | Migration Patterns | `migration-patterns.md` |
| Other/new patterns | Create new topic | New `.md` file |

### 3. Ask User Context

Present findings and ask:

```
🔍 **Recent Agent Activity Detected**

Files changed:
- `apps/worker/src/routes/batch.ts`
- `apps/pages/src/routes/batch.tsx`
- `apps/worker/src/handlers/image.ts`

Detected topic: **Batch Processing**

**What did the agent learn or discover?**
- Describe the pattern, fix, or insight
- Any edge cases discovered?
- New approaches or anti-patterns found?

Or reply with:
- "create topic [name]" for new topic
- "multiple topics" for splitting learnings
```

### 4. Structure & Capture

Format user input into entry:

```markdown
## [YYYY-MM-DD] [Agent Name] Session

### Task
[Brief description of what the agent worked on]

### What Was Learned
- Discovery 1
- Discovery 2
- Discovery 3

### Code References
- `path/to/file.ts` (line XX) - What this file demonstrates
- `path/to/another.ts` - Additional context

### Patterns to Follow
- [Pattern observed in code]
- [Another pattern]

### Anti-Patterns to Avoid
- [What NOT to do]
- [Common pitfalls discovered]

### Agent Context
- **Agent**: Codex / Other
- **Trigger**: User prompt description
- **Files modified**: List of key files

---

```

### 5. Write to Knowledge Base

Append to appropriate `docs/agent-knowledge/[topic].md`:

```bash
# If file exists, append
# If new topic, create with template
```

### 6. Summary Report

```
✅ **Learning Captured**

**Topic**: Batch Processing
**File**: `docs/agent-knowledge/batch-processing.md`
**Entries added**: 3 discoveries, 2 patterns

View: `cat docs/agent-knowledge/batch-processing.md`
```

## Knowledge Base Structure

```
docs/agent-knowledge/
├── README.md (index of all topics)
├── batch-processing.md
├── auth-sessions.md
├── image-generation.md
├── database-patterns.md
├── api-contracts.md
├── admin-panel.md
├── ui-components.md
├── migration-patterns.md
└── [new topics as discovered]
```

## Topic File Template

```markdown
# [Topic Name]

*Agent knowledge base for [topic]. Learnings from Codex and other external AI agents.*

---

## Quick Reference

### Key Patterns
- [Most important pattern]
- [Second most important]

### Common Pitfalls
- [What to avoid]
- [Common mistakes]

### Related Files
- `path/to/key/file.ts`
- `path/to/another/file.ts`

---

*Last updated: YYYY-MM-DD*

---

[Chronological entries below, newest first]

## [YYYY-MM-DD] [Agent] Session

[Entry content as above]

---
```

## Commands

### List all topics
```
"show all agent knowledge topics" → List `docs/agent-knowledge/` with summaries
```

### Search knowledge
```
"what does agent know about [topic]" → Grep in `docs/agent-knowledge/`
```

### Export topic
```
"export [topic] knowledge" → Output specific topic file content
```

### Create new topic
```
"create agent knowledge topic [name]" → New `.md` with template
```

## Best Practices

- **Be specific**: Capture exact patterns, not generalities
- **Include context**: Why was this approach taken? What constraints existed?
- **Link code**: Always include file paths and line references
- **Date everything**: Chronological view helps track evolution
- **Document failures**: Anti-patterns and pitfalls are as valuable as successes
- **Organize by feature**: Topic-based structure mirrors `docs/` organization

## Example Interaction

**User**: "capture learnings - codex just worked on batch regeneration"

**Agent Knowledge Skill**:
1. Runs `git diff HEAD~5..HEAD --stat`
2. Detects `apps/pages/src/routes/batch-regenerate.tsx` changed
3. Maps to topic: Batch Processing
4. Asks: "What did Codex learn about batch regeneration?"
5. User describes: "Found that regeneration reuses request IDs, need unique IDs for parallel safety"
6. Structures entry with code references to `batch-regenerate.tsx`
7. Appends to `docs/agent-knowledge/batch-processing.md`
8. Shows summary with file location

## Notes

- Agents output to stdout/stderr, so we rely on user-triggered capture
- Git diff analysis provides context for what was worked on
- Topic mapping is heuristic - user can override or create new topics
- Knowledge base complements `docs/` - this is for agent-specific learnings, not user documentation
- Always maintain chronological order within topic files (newest first)
