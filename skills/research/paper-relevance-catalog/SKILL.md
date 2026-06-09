---
name: paper-relevance-catalog
description: "Use when a research workflow needs a complete paper-by-paper citation appendix grouped by relevance, with a short summary for every paper so the user can inspect sources themselves."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [research, citations, literature-review, triage, appendix]
    related_skills: [alphaxiv-deep-research, arxiv]
---

# Paper Relevance Catalog

## Overview

Use this as the final companion step after a deep-research workflow. Its job is to produce a paper-by-paper source appendix that makes the search process inspectable by the user.

The key rule is explicit: **every searched paper that materially entered the research funnel should be listed with its citation, relevance bucket, and a short summary of what it is about.**

This companion skill exists so the final research output does not only show the best papers; it also shows what was considered and how those papers were judged.

## When to Use

Use when:
- a deep research task reviewed multiple papers
- the user wants source transparency
- the user may want to manually inspect the papers afterward
- you need an auditable trail of paper triage decisions

Do not use when:
- only one paper was inspected
- the user explicitly asks for a very short answer with no appendix
- the task is not paper-centric

## Required Rule

For the final output, include a dedicated section named something like:
- `Paper Relevance Catalog`
- `Searched Papers by Relevance`
- `Paper Triage Appendix`

That section must include **all searched papers in relevance categories**:
- **Super relevant**
- **Relevant**
- **Irrelevant**

If the search set is very large, include every paper that was actually opened, summarized, shortlisted, or otherwise materially evaluated. If some candidates were only glanced at by title and immediately discarded, say that explicitly and, if possible, include them in the Irrelevant bucket with a one-line reason.

Never omit inconvenient papers just because they weaken the narrative.

## Per-Paper Minimum Format

For every paper listed, include:
1. **Full citation handle** — title plus arXiv ID, DOI, or direct URL
2. **Relevance bucket** — Super relevant / Relevant / Irrelevant
3. **Short summary** — 1-3 sentences on what the paper is about
4. **Why it got this bucket** — 1 short reason tied to the user's question

Preferred compact format:

```markdown
- **Paper Title** — arXiv:2501.01234
  - **About:** Introduces a retrieval-augmented agent benchmark for long-horizon tasks.
  - **Why bucketed here:** Directly studies the same failure mode the user asked about.
```

## Bucket Definitions

### Super relevant
Use for papers that directly answer the user's question, benchmark the exact setting of interest, introduce a central method, or provide the strongest evidence.

### Relevant
Use for papers that are adjacent, supportive, comparative, historical, or partially applicable, but not central enough to anchor the answer.

### Irrelevant
Use for papers that appeared in discovery but are off-topic, too weakly related, use a materially different task setup, or fail the user's actual selection criteria.

Be honest. "Irrelevant" does not mean bad paper; it means not useful for this question.

## Standard End-of-Workflow Procedure

After the main synthesis is done:

1. gather the deduplicated set of searched / inspected papers
2. assign each paper to one of the three relevance buckets
3. write a short summary for each paper
4. provide a brief reason for the bucket assignment
5. append this catalog to the end of the answer

If the main answer already contains a shortlist table, do not treat that as sufficient. The catalog is a separate accountability artifact.

## Output Template

```markdown
## Paper Relevance Catalog

### Super relevant
- **Title** — arXiv/DOI/URL
  - **About:** ...
  - **Why bucketed here:** ...

### Relevant
- **Title** — arXiv/DOI/URL
  - **About:** ...
  - **Why bucketed here:** ...

### Irrelevant
- **Title** — arXiv/DOI/URL
  - **About:** ...
  - **Why bucketed here:** ...
```

## Quality Bar

Good catalogs are:
- complete enough to reconstruct the search funnel
- easy to skim
- explicit about why a paper matters or does not
- grounded in actual inspection, not guessed from titles alone

If you only know the title and abstract, say so implicitly in the wording and avoid overclaiming.

## Common Pitfalls

1. **Listing only the winning papers.** The whole point is to expose the search funnel, not just the final picks.
2. **Missing citation handles.** Every paper needs a traceable identifier or URL so the user can inspect it directly.
3. **Confusing "irrelevant" with "low quality."** The bucket is about fit to the question, not paper merit.
4. **Using vague summaries.** "This discusses agents" is too weak; say what task, method, or benchmark the paper is about.
5. **Failing to include the reason for bucketing.** The user should be able to understand the triage logic quickly.

## Verification Checklist

- [ ] Final answer contains a dedicated paper catalog section
- [ ] Papers are grouped into Super relevant / Relevant / Irrelevant
- [ ] Every listed paper has a citation handle or URL
- [ ] Every listed paper has a short "About" summary
- [ ] Every listed paper has a short reason for its bucket
- [ ] The catalog reflects the actual search funnel, not only final picks
