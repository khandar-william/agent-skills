---
name: stacked-prs
description: >-
  Split a large diff into stacked branches/PRs or rebase an existing stack onto
  latest main. Use only when the user explicitly invokes /stacked-prs with a
  subcommand (split current diff, rebase to latest main).
disable-model-invocation: true
---

# Stacked PRs

## Command routing

**If the user invokes only `/stacked-prs` (no subcommand):** show the help below and stop. Do not read diffs, run git commands, or propose a plan.

**If the user invokes `/stacked-prs split current diff`:** follow [Split current diff](#split-current-diff).

**If the user invokes `/stacked-prs rebase to latest main`:** follow [Rebase to latest main](#rebase-to-latest-main).

### Help (show when `/stacked-prs` has no subcommand)

```text
/stacked-prs — manage stacked pull requests

Commands:
  /stacked-prs split current diff   Propose (then execute) a stacked split of your current diff vs main
  /stacked-prs rebase to latest main   Rebase the entire stack onto origin/main with --update-refs

References:
  leveraging-llm-for-stacked-prs.md
  context-planning-small-prs.md
```

---

## Shared rules

- **Plan first (split only)** — do not create branches, commit, push, or open PRs until the user approves the split plan.
- **Never discard work** — no destructive git (`reset --hard`, `clean -fdx`, branch deletion, force-push) without explicit approval.
- **Backup before splitting** — save a recoverable snapshot; do not assume a safe branch exists.
- **Stage named files only** — never `git add .` / `git add -A`.
- **Linear stack** — each branch tip is an ancestor of the next; PRs target the branch below (first PR targets `main`).
- **Merge bottom-up** — reviewers merge PR #1 first, then #2, and so on.

Full references: [leveraging-llm-for-stacked-prs.md](leveraging-llm-for-stacked-prs.md), [context-planning-small-prs.md](context-planning-small-prs.md).

---

## Split current diff

The large change already exists (in `git diff`). Propose logical slices, then — after approval — create a **linear stack** of branches (one PR each).

### Splitting principles

Each PR is a **logical unit**, not an arbitrary ~300-line cut.

| Guideline | Detail |
|-----------|--------|
| Size target | Aim for under ~300 lines; split by meaning, not line count |
| CI | Every PR must compile, pass CI, and be reviewable in ~1 hour |
| Tests | Each PR carries tests for its behavior — do not defer all tests to the last PR |
| Balance | Too many PRs = maintenance burden; too few = hard review |

**Strategies** (pick what fits the diff):

| Strategy | Example stack |
|----------|----------------|
| By layer | migration → service → API → UI |
| By behavior | create → update → delete |
| Refactor first | pure refactor (zero behavior change) → feature |
| Interface first | contract + mocked tests → implementation |
| Mechanical isolation | dependency bump / formatting → logic changes |

Avoid: splitting too granularly (dead code with no context), mixing refactors with features, breaking the dependency chain.

### Phase 1 — Read the diff

Compare current work to `main` (committed **and** uncommitted):

```bash
git diff main...HEAD    # committed
git diff                # uncommitted
git status
```

Summarize slices you see. Check ownership signals (`CODEOWNERS`, repo equivalents) for natural reviewer boundaries.

### Phase 2 — Propose the plan (no git mutations yet)

For each branch/PR provide:

- **Branch name** — see naming convention below
- **PR title**
- **One-sentence purpose**
- **Files or paths** in this slice
- **Base branch** (`main` for first; previous branch for the rest)
- **Commit order** within the stack

Show the stack as a diagram (text tree or Mermaid). Ask the user to approve, merge thin slices, split oversized ones, and fix dependency order.

### Phase 3 — Execute (after approval)

1. Save backup:

```bash
SHA=$(git stash create "pre-split")
if [ -n "$SHA" ]; then
  git update-ref "refs/backup/pre-split-$(date +%s)" "$SHA"
fi
```

2. For each branch **bottom-up** (first targets `main`):
   - Create branch from its base
   - Stage **only** the files for this slice
   - Commit with a clear message
   - Push and open PR targeting the base branch

3. Report back: PR titles/URLs, anything left on the starting branch or working tree. Do not delete backup refs unless asked.

### Branch naming

```text
f/ or e/ or b/ + <prefix>-<index>-<slug>
```

Use numeric or letter indices (`0`, `1a`, `1b`, `2`, …) to preserve order. The **latest branch** (top of stack) contains every commit from all branches below it.

Example:

```text
main
 └── f/#17196-0-ctp10-db-migration              PR #1  (base: main)
      └── f/#17196-1a-ctp10-refactor-validator  PR #2  (base: 0)
           └── f/#17196-1b-ctp10-refactor-modifier   PR #3  (base: 1a)
                └── ...
                     └── f/#17196-z-ctp10-update-docs   PR #N (base: previous)
```

### What reviewers see

| Branch | PR targets | Diff shows |
|--------|------------|------------|
| `f/...-0-...` | `main` | Slice 0 only |
| `f/...-1a-...` | `0` | Slice 1a only (not everything below) |
| `f/...-z-...` | previous | Final slice only |

Each PR diff is **branch vs its base** — small and focused.

### Refining the plan

**Oversized slice** — split into two stacked branches, each delivering one reviewable behavior; keep tests with the behavior they cover.

**Isolate refactor** — pull moves/renames/zero-behavior-change into the first branch; feature logic starts in branch 2.

**Recover from bad split** — use `refs/backup/pre-split-*` and `git branch -v` / `git log --oneline`; do not destroy work.

### Split checklist

- [ ] Backup ref saved
- [ ] Split plan reviewed and approved
- [ ] Branches created bottom-up; each PR targets branch below
- [ ] Each PR compiles, passes CI, reviewable in ~1 hour

---

## Rebase to latest main

When `main` moves, rebase the entire stack from the **latest branch only** (Git 2.38+). Do **not** rebase each branch individually.

### Steps

1. Identify the stack — ask the user for branch names in order if not obvious from context.
2. Confirm the latest (top) branch.
3. Run:

```bash
git fetch origin
git checkout f/<prefix>-z-<slug>   # latest branch in stack
git rebase origin/main --update-refs
```

`--update-refs` moves every intermediate branch ref to the rebased commits.

4. On conflict: fix on the latest branch, then `git rebase --continue`. Finish the current rebase before fetching again.
5. Force-push all branches. GitHub allows at most **2 branch updates per push**:

```bash
git push -f -u origin 'f/...-0-...' 'f/...-1a-...'
git push -f -u origin 'f/...-1b-...' 'f/...-2-...'
# ... pairs until all branches pushed
```

Prefer `--force-with-lease` when pushing. Get explicit approval before force-pushing.

6. Remind the user to re-check CI on the bottom PR.

### Rebase checklist

- [ ] Stack is linear; names encode ticket + order
- [ ] Checked out latest branch only
- [ ] `git rebase origin/main --update-refs` completed
- [ ] Force-pushed all branches (2 per push)
- [ ] Re-check CI on bottom PR

---

## Common mistakes

| Mistake | Better approach |
|---------|-----------------|
| Rebasing every branch when `main` moves | Checkout latest only; `git rebase origin/main --update-refs` |
| `git add .` when creating a slice | Stage named files or hunks only |
| Splitting at 300 lines arbitrarily | Split by logical unit |
| Deferring all tests to last PR | Each branch carries its own tests |
| Branch targets `main` mid-stack | Every branch after first targets the branch directly below |
| Merging PRs out of order | Merge bottom-up |
| Forgetting force-push after rebase | Push all stack branches |
