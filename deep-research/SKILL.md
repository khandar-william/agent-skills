---
name: deep-research
description: "Produce deep research reports using a dual-agent Planner+Writer methodology inspired by Alibaba's WebWeaver. Dynamic outline, memory bank, section-by-section synthesis with inline citations. Use when the user asks for a deep dive, comprehensive analysis, long-form research report, or thorough investigation on any topic."
version: 1.0.0
tags: [research, deep-research, report-writing, webweaver, analysis]
---

# Deep Research Report Skill

## When to Use
- User asks for a "deep dive", "deep research", "comprehensive analysis", "thorough investigation"
- User wants a long-form research report (2,000–10,000+ words)
- Topic requires multi-source web research, synthesis, and structured output
- User explicitly mentions "deep research" or "research report"

## Methodology (WebWeaver-Inspired)

### Overview: Dual-Agent Pipeline

**Phase 1 — PLANNER**: Search, visit, extract, build a dynamic outline with citations.
**Phase 2 — WRITER**: Retrieve evidence per section, write analytically, cite sources.

In Hermes Agent, use `delegate_task` to run Phase 1 and Phase 2 as subagents, OR execute both phases in a single session for smaller reports. For large reports, the two-phase split keeps context manageable.

---

## Phase 1: PLANNER (Search + Outline Building)

### Step 1: Decompose the Query
- Analyze the user's question
- Identify 3–5 core sub-questions or facets to investigate
- Think about: causes, mechanisms, implications, comparisons, history, future

### Step 2: Search and Extract
For each sub-question, use the **two-track approach**:

**Track A — Known URLs first (always start here):**
1. **Construct** candidate URLs from domain knowledge: author sites, Wikipedia, known summary sites (Goodreads for books, official docs for tech topics, news outlets for current events)
2. **Fetch** each with `webclaw -f llm <url>` to check relevance
3. **Re-fetch promising pages** with `webclaw -f text <url>` to get full content (text mode returns 3-5x more content than llm mode)
4. **Follow leads** — URLs found inside fetched pages often point to deeper sources

**Track B — Search when you need to discover URLs:**
1. **Search** using the `web_search` tool (top-level agent tool, NOT available from `execute_code`)
2. If `web_search` is unavailable, use **browser** to navigate to search.brave.com and extract results from the page
3. **Select** the 2–3 most promising URLs from results
4. **Visit** each URL using `webclaw -f llm <url>` (preferred) or `web_extract`

**Extract** structured info from each page:
   - **Goal**: What specific info were you looking for?
   - **Evidence**: Full relevant passages (3+ paragraphs, preserve original text)
   - **Summary**: 1-paragraph synthesis of the page's contribution to the goal
5. **Store** in the Memory Bank (see below)

### Step 3: Build Dynamic Outline
After each batch of searches:
1. Draft or update a hierarchical outline (I. / A. / 1. / a.)
2. Go at least 3 levels deep
3. Attach source IDs to each subsection: `subsection title [1, 3, 7]`
4. **Iterate at least 3 times** — each round of searching should refine the outline
5. Merge subsections with similar content
6. Add subsections for gaps discovered during research
7. Ensure logical flow: context → analysis → implications → conclusion

### Step 4: Verify Completeness
- Every subsection must have at least one source citation
- If any subsection lacks citations → search specifically for that topic
- If outline has < 5 major sections → search for more facets
- If all sources are from one domain → diversify with more searches

---

## Memory Bank Structure

Maintain a running list of all visited sources. Each entry:

```
### Source [ID]: Title
- **URL**: https://...
- **Goal**: What info was sought
- **Evidence**: [Full relevant passages from the page]
- **Summary**: [1-paragraph synthesis]
```

Use this format throughout planning. During writing, you'll retrieve from this bank by ID.

---

## Phase 2: WRITER (Section-by-Section Synthesis)

### Step 1: Construct the Writing Prompt
Before writing, assemble:
- The research question
- The final outline (with source ID references)
- The full Memory Bank (all sources with evidence)

### Step 2: Write Section by Section
For each major section of the outline:
1. **Identify needed sources** — which IDs are cited in this section?
2. **Retrieve evidence** — pull the relevant passages from the Memory Bank
3. **Reason** — what insights can be drawn? What connections exist?
4. **Write** the section analytically (see Writing Standards below)
5. **Cite** inline using `[ID]` format

### Step 3: Assemble Final Report
Concatenate all sections. Append a References section at the end.

---

## Report Structure (Mandatory)

```
# [Title]

> **Key Takeaway**: [1–2 sentence TL;DR answering the question directly]

## I. [Major Section — Context/Background]
### A. [Subsection]
[Analytical prose with inline citations [1, 3]]

### B. [Subsection]
[More analysis]

## II. [Major Section — Core Analysis]
### A. [Subsection with table if applicable]
| Dimension | Option A | Option B | Option C |
|-----------|----------|----------|----------|
| ...       | ...      | ...      | ...      |

[Post-table analytical discussion — explain what the data reveals, why it matters]

### B. [Subsection]
[Deep analysis with citations]

## III. [Major Section — Implications/Future]
...

## IV. [Major Section — Conclusion/Synthesis]
[Synthesize findings, answer the original question directly]

---

## References
[1]. Source Title — https://...
[2]. Source Title — https://...
```

---

## Writing Standards

### Analytical Depth (NOT compilation)
- **Explain WHY**, not just WHAT
- Every claim needs: Evidence → Analysis → Implications
- Use analytical language: "This suggests that...", "The underlying mechanism is...", "Connecting these findings reveals..."
- **FORBIDDEN**: Listing facts without interpretation, direct copying, shallow paraphrasing

### Logical Argumentation
Each major point follows: **Claim → Evidence → Analysis → Implications**
- State the thesis
- Present supporting evidence with citation
- Explain how the evidence supports the claim
- Draw broader implications

### Citation Rules
- **MUST cite**: Statistics, data, attributed opinions, case studies, specific examples
- **MUST NOT cite**: Common knowledge ("The Earth is round")
- **Format**: Inline `[ID]` where ID matches the Memory Bank
- Every verifiable claim from sources gets a citation

### Tables
- Minimum 2 tables per report (for reports > 2,000 words)
- Use when: comparing options, presenting data, showing timelines, organizing categories
- Always follow with analytical discussion referencing the table

### Tone & Accessibility
- Assume reader has basic/no prior knowledge
- Define domain-specific terms when first introduced
- Bold key facts for scannability
- Short, descriptive, sentence-case headers
- Casual but precise language
- Self-contained — no need for external resources

### Length Guidelines
- Quick research: 1,500–3,000 words
- Standard deep research: 3,000–6,000 words
- Comprehensive deep dive: 6,000–10,000+ words
- Match depth to user's request; default to standard

---

## Execution Strategy for Hermes Agent

### Small Reports (< 3,000 words)
Single session, sequential execution:
1. Search 5–8 queries
2. Visit 8–12 pages
3. Build outline
4. Write report inline
5. Save to `~/research/YYYY-MM-DD-{slug}.md`

### Large Reports (3,000+ words)
**Preferred: direct execution** — run Phase 1 (search + extract) and Phase 2 (write) yourself. Subagent delegation for Phase 1 frequently returns empty/minimal results because the subagent doesn't execute multi-step tool chains reliably.

If you must delegate:
**Phase 1 subagent** (toolsets: `web`):
- Goal: "Research [topic] thoroughly. Search multiple angles, visit pages, extract evidence. Return: (1) a structured outline with source IDs, (2) a complete Memory Bank with all sources, evidence, and summaries."
- The subagent returns the outline + memory bank as its summary

**Phase 2 subagent** (toolsets: `file`):
- Goal: "Write a deep research report using the provided outline and memory bank. Follow the WebWeaver writing standards. Save to ~/research/YYYY-MM-DD-{slug}.md"
- Context: Pass the Phase 1 output (outline + memory bank)

### Always Save Output
Every deep research report MUST be saved to `~/research/YYYY-MM-DD-{slug}.md` where slug is a kebab-case summary of the topic.

---

## Search Strategy

### Query Decomposition Pattern
For topic "X", generate queries like:
- "X overview explained"
- "X latest research 2025 2026"
- "X vs alternatives comparison"
- "X challenges limitations problems"
- "X case studies real world examples"
- "X future trends predictions"
- "[specific subtopic of X] deep dive"

### Source Diversity
Aim for:
- At least 2 different source types (academic, news, blog, documentation)
- At least 3 different domains
- Mix of recent (last 6 months) and authoritative (established references)
- For technical topics: official docs + analysis pieces + community discussion
- **DuckDuckGo HTML search is broken** (returns empty as of 2026-06) — do not use as fallback

### Web Search Tools (in priority order)
1. **Direct URL construction** — Build URLs from domain knowledge (author sites, Wikipedia, known aggregators, news outlets). This is the most reliable approach and works from `execute_code`. Start here every time.
2. `web_search` — Brave Search via the built-in tool (works well, but only available as a top-level agent tool, NOT from `execute_code`)
3. **Browser search** — Navigate to search.brave.com via `browser_navigate` when you need to discover URLs and `web_search` is unavailable
4. `webclaw -f llm <url>` — Page reading with LLM analysis (preferred for quick orientation)
5. `webclaw -f text <url>` — Raw text extraction (use when `-f llm` returns too little or you need verbatim passages). Often returns much more content than `-f llm` (e.g., 45K chars vs 50K chars with full review text preserved).
6. `webclaw --crawl <url>` — Multi-page crawling for documentation sites
7. `web_extract <url>` — Fallback page extraction

---

## Pitfalls

- **Subagent delegation often fails for Phase 1**: With many models, delegating the research phase to a subagent returns minimal output (the subagent doesn't actually execute the searches). **Prefer direct execution** — run searches and page reads yourself in `execute_code` loops. Only delegate if you've verified the model handles multi-step tool chains reliably.
- **`web_search` not available from execute_code**: The `web_search` tool is a top-level agent tool, not a CLI command. It cannot be called from within `execute_code` scripts. Use `webclaw` with known URLs instead, or do browser-based search. When you need to discover URLs, try: (1) fetching a known aggregator (Goodreads search, Wikipedia), (2) using Brave Search via browser, or (3) constructing likely URLs from domain knowledge.
- **Brave Search via raw curl is blocked**: Returns the HTML search page, not results. Use the `web_search` tool or browser-based search.
- **DuckDuckGo HTML search is broken**: Returns empty HTML (no result links) as of 2026-06. Do not use as fallback for finding URLs. Use Brave Search via browser instead.
- **Source reliability patterns**: See `references/source-reliability.md` for an up-to-date table of which domains work vs. block scraping. Key known-good: Wikipedia, Goodreads (book pages), Amazon.sg, The Guardian, NirandFar. Key known-broken: DuckDuckGo HTML search (empty), Medium (Cloudflare), NYTimes (JS-required), NNGroup (blocks webclaw).
- **webclaw path**: `webclaw` is at `/home/ubuntu/.cargo/bin/webclaw` (in PATH). Use `-f llm` for analysis, `-f text` for raw extraction when llm mode returns too little.
- **webclaw `-f text` vs `-f llm` tradeoff**: `-f llm` produces a concise LLM summary (good for quick orientation, ~5-15K chars). `-f text` returns the full raw text (good for reviews, long-form content, verbatim quotes — often 20-50K+ chars). For book reviews, community comments, and detailed source material, prefer `-f text` to preserve original language. Use `-f llm` first to check if the page has relevant content, then switch to `-f text` for extraction.
- **Outline fossilization**: Don't lock the outline after first draft. Keep refining as you learn.
- **Source starvation**: If you have < 5 sources, search more. Quality degrades without evidence.
- **Context overflow**: For large reports, write section-by-section. Don't try to hold the entire report in context.
- **Shallow analysis**: Listing findings is NOT research. Every fact needs interpretation.
- **Missing citations**: Unsourced claims erode trust. When in doubt, cite.
- **Ignoring counterarguments**: Acknowledge limitations and alternative viewpoints.
- **Stale sources**: Prioritize recent information. Flag when using older sources.
- **Monoculture sources**: Don't rely on one domain or author. Diversify.

---

## Deliverables Beyond the Report
- **Mindmaps/Diagrams**: If the topic benefits from a visual overview, render a Mermaid mindmap as interactive HTML + PNG. See `references/mermaid-diagram-rendering.md` for the rendering workflow.

## Verification Checklist

Before delivering a report, verify:
- [ ] Key takeaway answers the question directly (1–2 sentences)
- [ ] Outline has 3+ major sections with subsections
- [ ] Every subsection has at least one citation
- [ ] At least 2 tables (for reports > 2K words)
- [ ] Analytical depth — every fact has interpretation
- [ ] References section lists all cited sources with URLs
- [ ] Report saved to ~/research/
- [ ] Self-contained — reader needs no external resources
