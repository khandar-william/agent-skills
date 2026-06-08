---
name: commit-message
description: >-
  Writes conventional commit messages from git diffs with bullet-point body
  details. Use when the user asks for a commit message, help committing staged
  changes, or summarizing a diff for git.
---

# Commit Message

Generate a conventional commit message from the code diff.

## Workflow

1. If no diff was provided, gather changes:
   - Staged: `git diff --cached`
   - Unstaged: `git diff`
   - Use staged changes when the user is about to commit; otherwise use what they specify.
2. Read the diff and infer the primary intent (feature, fix, refactor, docs, test, chore, etc.).
3. Match the repository's existing commit style when `git log` shows a clear convention; otherwise use [Conventional Commits](https://www.conventionalcommits.org/).

## Output Format

**Subject line** (required):

```
<type>(<optional scope>): <short imperative summary>
```

- **type**: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `ci`, `build`, or `style` (only when formatting-only).
- **scope**: optional module or area (e.g., `auth`, `api`).
- **summary**: imperative mood, ≤ 72 characters, no trailing period.

**Body** (required): bullet points describing *what* changed and *why*, not a line-by-line diff replay.

Example:

```
feat(auth): add idle session timeout configuration

- Add admin settings page for tenant idle timeout (5 min – 2 weeks)
- Validate input and apply default of 20,160 minutes on new tenants
- Invalidate sessions that exceed the new threshold on config change
```

## Rules

- One logical change per commit message; if the diff mixes unrelated work, say so and suggest splitting.
- Do not invent changes not present in the diff.
- Omit scope if unclear; do not guess issue IDs unless they appear in branch names or user context.
