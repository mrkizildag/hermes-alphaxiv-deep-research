# Hermes AlphaXiv Deep Research

A Hermes skill pack for citation-grounded literature review. The pack gives an agent inside Hermes a single, repeatable research loop and the fallbacks needed to run that loop whether or not AlphaXiv MCP is configured.

## What this is

The goal is narrow: take a research question, return a sourced answer plus a transparent paper-by-paper record of what the agent considered.

The pack is **free/public-first**. The workflow holds together with no API keys and no MCP servers configured — it falls back to public AlphaXiv pages, arxiv, and authenticated browser sessions. AlphaXiv MCP is treated as an upgrade path, not a hard dependency.

## The loop

Every skill in this pack contributes to one five-step loop.

1. **Discover.** Given a question, generate candidate papers. With AlphaXiv MCP, this is `discover_papers`. Without it, public alphaxiv search + arxiv listing.
2. **Read.** Pull full content for the top candidates. With MCP, `get_paper_content`. Without it, the public abstract/overview pages or the arxiv PDF.
3. **Verify.** Cross-check specific claims against the source text and against secondary references. With MCP, `answer_pdf_queries` for targeted evidence; otherwise web search and direct PDF reading.
4. **Synthesize.** Write a cited answer that distinguishes "directly supported by paper X" from "inferred" from "still uncertain".
5. **Catalog.** Append a relevance table that lists every paper the agent considered, why each was kept or dropped, and where it ended up cited.

The catalog step is the part most research assistants skip. It is what makes the output auditable.

## Example

You ask:

> What does the recent literature say about test-time compute scaling laws?

You get back:

- a synthesized answer with inline citations to specific papers
- a relevance catalog table at the end, shaped like:

| Paper | Status | Why |
|---|---|---|
| Snell et al. 2024 — *Scaling Test-Time Compute...* | cited | direct empirical scaling results |
| Wu et al. 2024 — *Inference Scaling Laws* | cited | complementary theoretical bounds |
| Author 2023 — *Older retrieval baseline* | considered, dropped | superseded by a 2024 follow-up |

No silent filtering. Whatever the agent looked at shows up in the table.

## The skills

| Skill | Layer | Role in the loop | Triggers when |
|---|---|---|---|
| `alphaxiv-deep-research` | research | Orchestrator. Drives Discover → Read → Verify → Synthesize end-to-end, calling the other skills as needed. | User asks for a deep, multi-source review on a topic where AlphaXiv coverage is plausible. |
| `paper-relevance-catalog` | research | Produces the final Catalog step. | The loop has finished discovery and synthesis and you need a defensible record of every paper considered. |
| `authenticated-browser-research` | research | Read + Verify fallback when public web tools are not enough. | A persistent authenticated browser session is available, or the target site has auth-gated content. |
| `browser-session-api-discovery` | productivity | Underpins authenticated Read/Verify by reusing the app's own API instead of scraping. | The agent needs structured data out of a site the user is already signed into. |
| `browser-auth-walkthroughs` | productivity | Underpins Read/Verify by handling the actual sign-in step. | The agent needs to log in to a site before it can read anything useful. |

`alphaxiv-deep-research` is the entry point; the others are called from it (directly or via `authenticated-browser-research`).

## When AlphaXiv MCP is unavailable

The loop does not change. Only the backbone for each step changes.

| Step | With AlphaXiv MCP | Without |
|---|---|---|
| Discover | `discover_papers` | public alphaxiv search + arxiv listing |
| Read | `get_paper_content` | public overview/abstract pages, arxiv PDF |
| Verify | `answer_pdf_queries` | web search + direct PDF reading |
| Synthesize | same | same |
| Catalog | same | same |

If only the browser is authenticated (no MCP), `authenticated-browser-research` handles Read and Verify against the live alphaxiv UI.

## Install

From GitHub:

```bash
hermes profile install github.com/mrkizildag/hermes-alphaxiv-deep-research --alias
```

From a local checkout:

```bash
hermes profile install /path/to/hermes-alphaxiv-deep-research --name research-skills-test
```

## Optional: AlphaXiv MCP setup

To enable the MCP backbone for Discover/Read/Verify:

```bash
hermes mcp add alphaxiv --url https://api.alphaxiv.org/mcp/v1 --auth oauth
hermes mcp login alphaxiv
hermes mcp test alphaxiv
```

OAuth 2.1 + PKCE is handled by Hermes. Tokens persist at `~/.hermes/mcp-tokens/alphaxiv.json`.

## Repo layout

- `skills/research/` — the three research skills (`alphaxiv-deep-research`, `authenticated-browser-research`, `paper-relevance-catalog`)
- `skills/productivity/` — the two browser helper skills used by the research skills
- `docs/skill-graph.md` — the explicit related-skill graph
- `distribution.yaml` — Hermes profile manifest

Hermes-bundled skills that are referenced but not vendored here: `arxiv`, `dogfood`, `hermes-agent-skill-authoring`. They already ship with Hermes.
