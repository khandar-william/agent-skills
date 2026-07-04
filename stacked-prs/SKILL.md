---
name: stacked-prs
description: >-
  Split a large diff into stacked branches/draft PRs or rebase an existing stack onto
  latest main. Use when the user invokes /stacked-prs with split or rebase intent
  (natural language is fine; verbatim subcommands are optional).
disable-model-invocation: true
---

# Stacked PRs

## Command routing

Parse **intent** from the full `/stacked-prs …` message. Subcommands are shorthand, not required.

| Intent | Route to | Example invocations |
|--------|----------|---------------------|
| **Help** | Show [help](#help) and stop | `/stacked-prs` alone, or only "help" / "what can you do?" |
| **Split** | [Split current diff](#split-current-diff) | `split current diff`; "break this into stacked PRs"; "split my large commit"; "stack these changes"; "I need stacked PRs for commit `6a2d416`" |
| **Rebase** | [Rebase to latest main](#rebase-to-latest-main) | `rebase to latest main`; "rebase my stack"; "update the stack onto latest main"; "main moved, refresh the stack" |

**Split signals:** split, break up, stack, slice, separate PRs, too large for one PR, stacked PRs.

**Rebase signals:** rebase, refresh, update stack, main moved, sync with main, `--update-refs`.

If both intents appear, ask which one to run. If intent is unclear after reading the message, show help.

**Do not** read diffs, run git commands, or propose a split plan for help-only invocations.

### Help (show when intent is help-only)

```text
/stacked-prs — manage stacked pull requests

Split (natural language OK):
  /stacked-prs split my changes into stacked PRs
  /stacked-prs break this large commit into stacked PRs <base-ref>
  /stacked-prs stack these changes since origin/main

Rebase (natural language OK):
  /stacked-prs rebase my stack onto latest main
  /stacked-prs main moved — update the stack

Optional split base: name a commit, branch, or tag instead of main (see Split → Resolve refs).

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
- **Linear stack** — each branch tip is an ancestor of the next; PRs target the branch below (first PR targets the [split base](#resolve-refs), default `main`).
- **Merge bottom-up** — reviewers merge PR #1 first, then #2, and so on.

Full references: [leveraging-llm-for-stacked-prs.md](leveraging-llm-for-stacked-prs.md), [context-planning-small-prs.md](context-planning-small-prs.md).

---

## Split current diff

The large change already exists in git history and/or the working tree. Propose logical slices, then — after approval — create a **linear stack** of branches (one PR each).

### Resolve refs

Before reading the diff, determine **split base** and **split tip** from the user message and repo context.

| Ref | Default | When user specifies |
|-----|---------|---------------------|
| **Split base** | `main` (prefer `origin/main` if it exists and differs) | Any commit SHA, branch, or tag they name as base / since / from / onto / "after" |
| **Split tip** | `HEAD` (includes uncommitted work) | A named commit ("this commit `abc123`", "break up `abc123`") when they mean one commit only |

**Interpretation rules:**

1. **Range split (most common):** user wants everything after a point → `git diff <split-base>...<split-tip>`. Example: `/stacked-prs break this large commit into stacked PRs 6a2d416` → base `6a2d416`, tip `HEAD`.
2. **Single-commit split:** user clearly points at one commit to decompose → use `git show <sha>` and/or `git diff <sha>^..<sha>`; first PR still targets split base (`<sha>^` or an explicit base they gave).
3. **Ambiguous SHA:** if "this commit `sha`" could mean base or tip, state your reading in the plan and ask for confirmation before executing.

Verify refs exist (`git rev-parse --verify <ref>`). Say which base and tip you used in the split plan.

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

Compare current work to the resolved split base (committed **and** uncommitted unless tip is a fixed commit):

```bash
git diff <split-base>...<split-tip>   # committed range
git diff                              # uncommitted (skip if tip is not HEAD)
git status
git log --oneline <split-base>..<split-tip>   # commits in range
```

Summarize slices you see. Check ownership signals (`CODEOWNERS`, repo equivalents) for natural reviewer boundaries.

### Phase 2 — Propose the plan (no git mutations yet)

State **split base** and **split tip** at the top of the plan.

For each branch/PR provide:

- **Branch name** — see naming convention below
- **PR title**
- **One-sentence purpose**
- **Files or paths** in this slice
- **Base branch** (split base for the first PR; previous branch for the rest)
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

2. For each branch **bottom-up** (first targets split base):
   - Create branch from its base
   - Stage **only** the files for this slice
   - Commit with a clear message
   - Push the branch
   - Generate the PR description (see [PR descriptions](#pr-descriptions))
   - Open a **draft** PR targeting the base branch (see [Opening PRs](#opening-prs))

3. Report back: PR titles/URLs, split base/tip used, anything left on the starting branch or working tree. Do not delete backup refs unless asked.

### PR descriptions

Before opening each PR, get the slice diff:

```bash
git diff <base-branch>...<branch>
```

**If a `pr-explainer` skill exists** — check your assigned skills, or look for `SKILL.md` at `~/.agents/skills/pr-explainer/`, `~/.cursor/skills/pr-explainer/`, or `.cursor/skills/pr-explainer/`. When found, read that skill and generate the PR body from the slice diff using its template and guidelines.

**Otherwise** — write a short fallback body: one-sentence overview, bullet list of key changes, and a brief test plan.

Add stack context when helpful (e.g. "PR 2 of N in a stack; merge bottom-up" or when split base is not `main`).

### Opening PRs

Always create **draft** PRs so reviewers are not notified until the stack is ready:

```bash
gh pr create --draft \
  --base <base-branch> \
  --head <branch> \
  --title "<title>" \
  --body "$(cat <<'EOF'
<description>
EOF
)"
```

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

When split base is not `main`, PR #1 still targets that ref (branch or commit); say so in the PR description.

### What reviewers see

| Branch | PR targets | Diff shows |
|--------|------------|------------|
| `f/...-0-...` | split base | Slice 0 only |
| `f/...-1a-...` | `0` | Slice 1a only (not everything below) |
| `f/...-z-...` | previous | Final slice only |

Each PR diff is **branch vs its base** — small and focused.

### Refining the plan

**Oversized slice** — split into two stacked branches, each delivering one reviewable behavior; keep tests with the behavior they cover.

**Isolate refactor** — pull moves/renames/zero-behavior-change into the first branch; feature logic starts in branch 2.

**Recover from bad split** — use `refs/backup/pre-split-*` and `git branch -v` / `git log --oneline`; do not destroy work.

### Split checklist

- [ ] Split base and tip resolved and stated
- [ ] Backup ref saved
- [ ] Split plan reviewed and approved
- [ ] Branches created bottom-up; each PR targets branch below
- [ ] Each PR description generated (via `pr-explainer` when available)
- [ ] Each PR opened as a **draft**
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
git push --force-with-lease -u origin 'f/...-0-...' 'f/...-1a-...'
git push --force-with-lease -u origin 'f/...-1b-...' 'f/...-2-...'
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
| Assuming split base is always `main` | Resolve base from the message; default to `main` only when unspecified |
| Merging PRs out of order | Merge bottom-up |
| Forgetting force-push after rebase | Push all stack branches |
