---
name: pr-explainer
description: >-
  Generates detailed Pull Request descriptions from code diffs to ease human
  review. Use when the user provides a diff or asks for a PR description, PR
  summary, or explanation of code changes.
---

# PR Explainer

A skill for generating clear and helpful Pull Request descriptions from code diffs.

## Instructions

When the user asks you to write a Pull Request description based on a code diff, follow these guidelines to create a high-quality explanation that makes the reviewer's job easier:

1. **Analyze the Diff**: Carefully read through the provided code diff to understand the scope and intent of the changes. Identify the files modified, new features added, bug fixes, or refactoring performed.

2. **Structure the Description**: Use a clear, standardized format for the Pull Request description to ensure all necessary context is provided. Unless the user specifies a different template, ALWAYS use the following structure:

### PR Description Template

```markdown
## Overview
[Provide a concise, 1-2 sentence high-level summary of what this Pull Request accomplishes and why the changes were made.]

## Changes Made
[Detail the specific modifications made in the code. Use bullet points and group them logically (e.g., by feature or file) if there are many changes.]
- [Change 1: e.g., Added authentication middleware]
- [Change 2: e.g., Refactored user creation logic]

## Motivation
[Explain the reasoning behind these changes. What problem does this PR solve? Does it address a specific issue or feature request?]

## Notes for the Reviewer
[Provide any additional context that might help the reviewer. Point out specific areas where you'd like feedback, mention potential edge cases, or explain complex logic decisions.]
```

3. **Be Specific but Accessible**: Translate code-level changes into human-readable explanations. Avoid just reading the diff back to the user; instead, explain the *impact* and *purpose* of the changes.

4. **Review and Refine**: Before presenting the final description, review it to ensure it accurately reflects the diff and maintains a professional, helpful tone. If the diff is too large or lacks obvious context, you may ask the user for a brief summary of their intent to improve the PR description.
