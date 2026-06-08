# WebWeaver Extraction Prompt Pattern

Use this pattern when extracting information from visited web pages during the planning phase.

## Page Extraction Prompt

When you visit a page and need to extract research-relevant information, structure your extraction as:

```
Given the following webpage content and research goal, extract relevant information:

## Webpage Content
{page_content}

## Research Goal
{goal}

## Instructions
1. **Rationale**: Locate the specific sections/data directly related to the research goal
2. **Evidence**: Extract the most relevant information — output the FULL original context, at least 3 paragraphs
3. **Summary**: Organize into a concise paragraph with logical flow, assessing the contribution to the goal
```

## Output Format (for Memory Bank)

```json
{
  "url": "https://...",
  "goal": "What specific information was sought",
  "evidence": "Full relevant passages from the page (preserve original text, 3+ paragraphs)",
  "summary": "Concise synthesis paragraph — what this source contributes to the research"
}
```

## Key Principles
- **Never compress evidence during extraction** — preserve full original text
- **Evidence is for later retrieval** — the writer needs the raw material, not summaries of summaries
- **Summary is for the outline** — quick orientation for what this source covers
- **Goal grounds the extraction** — prevents collecting tangential information
