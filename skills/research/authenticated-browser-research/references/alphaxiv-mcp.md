# AlphaXiv API / MCP Notes

Condensed notes from evaluating AlphaXiv as a Hermes-backed deep-research surface.

## Browser-side capability split

Verified publicly accessible in the browser:
- homepage / explore
- paper pages
- overview/blog pages
- audio + transcript pages
- similar papers pane
- comments UI surface

Verified gated at execution time:
- assistant/research-agent UI is visible without login, but attempting to send a question triggers an account gate (`Create an account to use our research agent and more!`).

Practical lesson: distinguish **UI visible without login** from **agent execution allowed without login**.

## Public HTTP API surface that worked anonymously

Useful anonymous endpoints observed from the OpenAPI surface and live requests:
- `GET /v2/papers/{upid}/metadata`
- `GET /search/v2/paper/fast?q=...&includePrivate=false`
- `GET /papers/v3/{id}/similar-papers`
- `GET /papers/v3/{paperVersion}/overview/{language}`
- `GET /papers/v3/{id}/preview`

These are enough for lightweight paper lookup, overview retrieval, and related-work discovery even before auth.

## Auth-required API / agent surface

Authenticated endpoints returned `401 Missing Authorization` when probed without a token, including:
- `GET /assistant/v2`
- `GET /users/v3`
- `GET /mcp/v1`

The OpenAPI surface also exposes assistant/chat endpoints such as:
- `POST /assistant/v2/chat`
- `GET /assistant/v2/{llmChat}/messages`
- trace / usage / feedback endpoints

Takeaway: AlphaXiv has a real backend for research-agent workflows, but anonymous access is limited to public paper/search features.

## MCP entry points

Verified URLs:
- docs: `https://www.alphaxiv.org/docs/mcp`
- MCP server: `https://api.alphaxiv.org/mcp/v1`
- protected-resource metadata: `https://api.alphaxiv.org/.well-known/oauth-protected-resource/mcp/v1`
- auth server: `https://clerk.alphaxiv.org`

The MCP docs describe 4 tools:
- `discover_papers`
- `get_paper_content`
- `answer_pdf_queries`
- `read_files_from_github_repository`

This is a strong fit for Hermes-driven research loops: discovery → paper extraction → targeted PDF Q&A → repo inspection.

## Hermes integration path

Treat AlphaXiv as a **remote OAuth-authenticated MCP server**, not a static API-key integration.

Recommended config shape:

```yaml
mcp_servers:
  alphaxiv:
    url: "https://api.alphaxiv.org/mcp/v1"
    auth: oauth
```

Recommended commands:

```bash
hermes mcp add alphaxiv --url https://api.alphaxiv.org/mcp/v1 --auth oauth
hermes mcp login alphaxiv
hermes mcp test alphaxiv
```

Hermes docs confirm for `auth: oauth` HTTP MCP servers:
- OAuth 2.1 + PKCE is handled automatically
- browser auth opens on first connect when possible
- tokens persist at `~/.hermes/mcp-tokens/<server>.json`
- headless/paste-back OAuth is supported by pasting the final redirect URL or `?code=...&state=...`

## Setup pitfall

If Hermes MCP commands exist but runtime MCP support is missing, install the Python MCP SDK into the active Hermes environment first. In a Hermes-managed venv, `uv pip install mcp` is the preferred default because it respects the current `VIRTUAL_ENV`.

## Recommendation heuristic

For AlphaXiv specifically:
1. use browser tools to verify public vs gated product behavior,
2. use anonymous HTTP endpoints for lightweight paper discovery/metadata,
3. prefer AlphaXiv's hosted MCP server for the full Hermes deep-research workflow,
4. only fall back to scraping the assistant UI if MCP/OAuth integration is unavailable.
