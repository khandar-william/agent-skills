# For Code Authors: Planning Small PRs

Lumping thousands of lines of code changes into a single PR is considered bad practice. To facilitate a fast and effective human review process, large changes should be split into multiple, self-contained, small PRs.

This practice is known as [Stacked PRs](https://graphite.com/blog/stacked-prs).

## Guidelines for Splitting PRs

1. **Size target:** Each PR should aim for under 300 lines of change.  
   1. This is not a hard limit, but an initial target to keep in mind.  
   2. In some cases, splitting a PR too small will cause the changes to lose context or coherence.  
2. **Number of PRs:** Strike a balance between "too many PRs" and "just one big PR."  
   1. Too many, and the author will spend excessive time keeping all PR branches up to date.  
   2. Too few, and the reviewer will struggle to understand the code.  
3. **CI must pass:** Every small PR is still required to pass all CI checks.

## How to Split PRs Effectively

Splitting PRs is not about arbitrarily cutting code at the 300-line mark. Each PR should represent a **logical unit of change** — something that makes sense on its own, can be reviewed independently, and leaves the codebase in a working state.

Here are practical strategies you can apply to break down large changes into focused, reviewable PRs:	

### Split by Layer or Boundary

When building a feature that touches multiple layers (e.g., database, backend service, API endpoint, frontend), split along those boundaries:

1. **PR 1:** Database migration and model/entity changes.  
2. **PR 2:** Service/business logic layer that uses the new model.  
3. **PR 3:** API endpoint or controller that exposes the new logic.  
4. **PR 4:** Frontend or client-side integration.

Each PR builds on the previous one, but each is independently reviewable and testable.

### Split by Behavior

If a single feature involves multiple behaviors or use cases, submit one PR per behavior:

1. **PR 1:** Implement the "create" flow.  
2. **PR 2:** Implement the "update" flow.  
3. **PR 3:** Implement the "delete" flow with soft-delete logic.

This makes it easy for reviewers to verify each behavior against the corresponding PRD scenario.

### Separate Refactoring and Mechanical Changes from Feature Work

Never bundle refactors or mechanical changes with new features. These are fundamentally different types of work and should be reviewed independently.

**Refactoring** — restructuring existing code to improve design, readability, or maintainability — must result in **zero behavior change**. If you need to restructure code before building on it:

1. **PR 1:** Pure refactor — move, rename, or restructure code. Tests should continue to pass as-is.  
2. **PR 2:** New feature built on top of the refactored code.

This separation makes rollback straightforward: if the feature has a problem, you revert PR 2 without losing the cleanup from PR 1\.

**Mechanical changes** — dependency upgrades, formatting fixes, import reordering, or large-scale renaming across files — follow the same principle. These are typically low-risk but high-noise; mixing them with logic changes makes it nearly impossible for a reviewer to spot what actually matters. Always isolate them into their own PRs.

In both cases, the reasoning is the same: when a PR mixes intent (cleanup *and* new behavior), reviewers cannot confidently assess either one, and rollback becomes risky because the changes are entangled.

### Introduce Interfaces Before Implementations

When adding a new integration or dependency (e.g., a new external service client), consider splitting it as:

1. **PR 1:** Define the interface/contract, data models, and write tests against the interface (using mocks or stubs).  
2. **PR 2:** Implement the concrete integration behind the interface.

This allows the reviewer to evaluate the design and contract first, then focus purely on implementation details in the follow-up.

## What a Good Small PR Looks Like

A well-split PR should meet these criteria:

1. **Self-contained:** It compiles, passes CI, and does not leave the codebase in a broken state.  
2. **Single purpose:** A reviewer can summarize what the PR does in one sentence.  
3. **Testable:** It includes tests that cover the new or changed behavior introduced in that PR.  
4. **Reviewable in under 1 hour:** If a reviewer needs significantly more time, the PR is likely too large or too complex.

## Common Mistakes When Splitting

1. **Splitting too granularly:** A PR that only adds a single data class with no usage or tests provides no reviewable context. Each PR should deliver a meaningful slice of progress.  
2. **Leaving dead code:** Splitting sometimes tempts authors to merge code that is not yet called anywhere. If you must do this, clearly mark it and ensure the follow-up PR that wires it in is submitted promptly.  
3. **Breaking the dependency chain:** Each PR in a stack must be mergeable on its own. If PR 3 cannot work without PR 2 being merged first, make that dependency explicit in the PR description and keep the stack in order.  
4. **Forgetting to update tests across the stack:** Each PR must have its own passing tests. Do not defer all testing to the "final" PR in a stack — that defeats the purpose of splitting.