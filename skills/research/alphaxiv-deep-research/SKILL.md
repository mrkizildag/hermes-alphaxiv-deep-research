---
name: alphaxiv-deep-research
description: "Use when the user wants a deep research workflow in Hermes built around AlphaXiv MCP, with browser/web fallbacks and citation-grounded synthesis."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [research, alphaxiv, mcp, literature-review, synthesis]
    related_skills: [arxiv, authenticated-browser-research, browser-session-api-discovery, paper-relevance-catalog]
---

# AlphaXiv Deep Research

## Overview

Use AlphaXiv's hosted MCP server as the primary research backbone inside Hermes, then combine it with Hermes' built-in web, browser, terminal, file, and delegation tools for a full deep-research loop.

Best pattern:
1. discover candidate papers,
2. pull paper content,
3. ask targeted PDF questions,
4. inspect linked GitHub repos when relevant,
5. synthesize with explicit citations and confidence notes,
6. run the `paper-relevance-catalog` companion skill at the end to expose the search funnel,
7. fall back to browser/web/arXiv when MCP is unavailable or public-only access is enough.

## When to Use

Use when the user wants:
- a literature review or research brief
- a Feynman-style deep-research workflow in Hermes
- paper comparison across a topic, method, benchmark, or author set
- paper + codebase analysis
- a citation-grounded answer rather than a quick web summary

Do not use as the first choice when:
- the user only needs one quick fact from one paper
- the topic is not paper-centric
- AlphaXiv MCP is unavailable and plain web search is clearly enough

## Preconditions

AlphaXiv MCP should be configured and authenticated in Hermes:

```yaml
mcp_servers:
  alphaxiv:
    url: "https://api.alphaxiv.org/mcp/v1"
    auth: oauth
```

Useful checks:

```bash
hermes mcp list
hermes mcp test alphaxiv
```

Expected AlphaXiv MCP tools:
- `mcp_alphaxiv_discover_papers`
- `mcp_alphaxiv_get_paper_content`
- `mcp_alphaxiv_answer_pdf_queries`
- `mcp_alphaxiv_read_files_from_github_repository`

If the current chat session cannot see newly added MCP tools, start a fresh session or reload MCP in the active runtime.

## Tool Selection Heuristic

### Prefer AlphaXiv MCP for
- broad paper discovery with semantic retrieval
- extracting structured paper content
- targeted questions against one paper PDF
- reading a paper's GitHub repository

### Prefer Hermes web/browser tools for
- verifying product/UI behavior
- checking public AlphaXiv pages, audio transcripts, comments, similar papers
- grabbing ancillary sources outside AlphaXiv
- handling cases where the MCP path is unavailable in the current session

### Prefer arXiv / Semantic Scholar style fallbacks for
- simple paper lookup without auth
- supplementary citation or recommendation checks
- redundancy when AlphaXiv misses obvious candidates

## Standard Workflow

### 1. Scope the research question
Rewrite the user request into:
- topic
- decision to support
- time horizon or recency preference
- output format
- required depth

If no depth is specified, default to a concise but evidence-backed brief with 5-10 papers.

### 2. Discover papers
Use `mcp_alphaxiv_discover_papers` first.

Good input pattern:
- `keywords`: 3-4 short exact terms
- `question`: a richer semantic brief with methods, tasks, benchmarks, failure modes, or applications
- `difficulty`: 3 for focused lookup, 6-8 for broad literature discovery

Run multiple discovery passes when recall matters, for example:
- method framing
- benchmark framing
- application framing
- competing terminology / aliases

Deduplicate results by arXiv ID or canonical URL.

### 3. Pull content for the shortlist
For the top papers, use `mcp_alphaxiv_get_paper_content`.

Default behavior:
- start with the structured AI-generated report
- switch to `fullText=true` only when the report is insufficient or you need raw text

Capture for each paper:
- citation identity
- core claim
- method/setup
- datasets/benchmarks
- main quantitative results
- limitations
- relevance to the user's actual question

### 4. Ask targeted PDF questions
Use `mcp_alphaxiv_answer_pdf_queries` for precise evidence extraction.

Batch all important questions for the same paper in one call, such as:
- evaluation metrics and datasets
- main ablations
- failure modes
- compute or latency constraints
- limitations or open problems

Use the returned page-level text directly when constructing citations.

### 5. Inspect codebases when implementation matters
If the paper has code and the user's question touches implementation, use `mcp_alphaxiv_read_files_from_github_repository`.

Efficient pattern:
1. read `/` for repo overview,
2. identify likely entrypoints,
3. read only the relevant files/directories,
4. verify whether the code supports the paper claims you cite.

### 6. Synthesize carefully
Final outputs should separate:
- strongly supported claims
- plausible but weakly supported inferences
- unresolved contradictions

Recommended answer structure:
1. direct answer / executive summary
2. comparison table of key papers
3. evidence-backed takeaways
4. caveats / disagreement points
5. recommended next papers or codebases

### 7. Run the companion catalog step
At the end of the deep-research answer, run the `paper-relevance-catalog` companion skill.

This is mandatory when multiple papers were searched or evaluated.

The companion output must:
- include explicit citation handles for all materially searched papers
- group them into `Super relevant`, `Relevant`, and `Irrelevant`
- include a short summary of what each cited paper is about
- include a short reason why each paper belongs in that bucket

The purpose is to let the user inspect the search funnel themselves rather than only seeing the final shortlist.

## Fallback Workflow When MCP Is Missing

If AlphaXiv MCP is unavailable in the current session:

1. use `web_search` for discovery
2. use the `arxiv` skill and direct arXiv URLs for lookup
3. use browser/public AlphaXiv pages for overview/audio/similar
4. use `web_extract` when available for public pages and PDFs
5. use terminal + public APIs where appropriate

Important: distinguish between AlphaXiv features that are publicly visible and assistant actions that are auth-gated.

## Parallelization Pattern

For larger reviews, delegate independent paper analyses in parallel.

Good split:
- child 1: methods cluster A
- child 2: methods cluster B
- child 3: codebase inspection for the most promising paper

Require each child to return:
- paper IDs / URLs
- concrete claims
- evidence snippets or page references
- unresolved questions

Verify important claims in the parent before presenting them as facts.

## Output Standards

Always aim for:
- explicit paper titles or arXiv IDs
- quoted or page-grounded evidence when making detailed claims
- clear separation of summary vs evidence
- no invented numbers, benchmarks, or citations

When evidence is thin, say so plainly.

## Common Pitfalls

1. **Using only one discovery query.** Important papers are often missed unless you vary wording.
2. **Summarizing from memory instead of evidence.** Pull content and targeted PDF excerpts before synthesizing.
3. **Over-reading the AI-generated overview.** Use raw PDF query extraction when details matter.
4. **Treating public AlphaXiv UI as proof of executable agent access.** Assistant UI may be visible while action is auth-gated.
5. **Skipping code inspection when the user's question is implementation-sensitive.** Claims about training/inference behavior may only be clear in the repo.
6. **Failing to restart or refresh session state after MCP changes.** The server can be healthy while the current chat still lacks the tools.

## Verification Checklist

- [ ] AlphaXiv MCP is enabled and testable
- [ ] Discovery used more than one framing when recall matters
- [ ] Key papers were read via content extraction, not title-only skim
- [ ] Detailed claims were checked with targeted PDF queries
- [ ] Code was inspected when implementation questions mattered
- [ ] Final answer distinguishes evidence from inference
- [ ] Paper IDs / URLs are included for traceability
