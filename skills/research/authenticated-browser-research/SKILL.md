---
name: authenticated-browser-research
description: Use a persistent authenticated browser session for research on sites with better in-browser discovery/synthesis than public APIs; verify free/public vs gated capabilities before promising workflows.
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [research, browser, authenticated-session, camofox, web]
    related_skills: [browser-auth-walkthroughs, browser-session-api-discovery, arxiv]
---

# Authenticated Browser Research

Use this skill when research depends on a live browser session rather than plain API/search access — especially when the target site has stronger discovery, summarization, or personalization features in the web UI than in public endpoints.

Typical triggers:
- The user mentions a persistent browser session (for example Camofox) is already running
- The target site may expose some features publicly and reserve others for login or paid tiers
- The user wants to know whether a research tool is usable "for free" or only through account-gated features
- The site is dynamic enough that browser inspection is more reliable than generic search snippets

## Core workflow

1. **Prefer the live browser session when the user points you to one.**
   If the environment has a persistent authenticated browser session, use browser tools first for feature verification and workflow discovery.

2. **Separate public access from premium access.**
   Verify these independently:
   - landing page loads without login
   - search/discovery UI is visible
   - opening individual items/pages works unauthenticated
   - advanced controls that imply gating (Sign In, Upgrade to Pro, plan/credits UI, personalization, etc.)

3. **Describe capabilities in three buckets:**
   - clearly public and verified
   - likely gated / premium-signaled
   - not yet verified end-to-end

4. **Do not overclaim.**
   If you can browse and open content pages but have not executed the full agentic workflow, say so directly. Distinguish "I can access the site" from "I verified the strongest research features work for free".

5. **Use browser DOM inspection to confirm what the snapshot implies.**
   Useful checks:
   - `document.body.innerText`
   - visible buttons/links containing words like `Sign In`, `Upgrade`, `Pro`, `Pricing`, `Credits`, `Plan`
   - whether search/input widgets are present and enabled

6. **Probe action-triggered gates, not just visible UI.**
   A research site may expose an impressive assistant panel publicly while gating actual execution only after interaction. When evaluating a "research agent" workflow:
   - open the assistant/chat panel if it is visible
   - type a harmless test prompt or click the first action needed to start a run
   - record whether the site allows execution, requests login, or requests payment/upgrade
   - separately test adjacent features like `Similar`, `Comments`, `Notes`, and `Audio` because they may have different auth rules

   Report these separately as:
   - **UI visible without login**
   - **execution allowed without login**
   - **execution blocked by account/pro gate**

## Credential/env verification pitfall

When checking whether a research integration is configured:
- Avoid claiming keys exist based only on redacted terminal output.
- Hermes may block direct `read_file` access to `~/.hermes/.env` because it is a credential store.
- If you must inspect from terminal, parse only variable names and whether values are empty/set; do **not** rely on redacted echoes of full lines.
- Distinguish:
  - values present in the file
  - values present in the current process environment
  - values required by an optional CLI/tool that is not yet installed

## API / MCP follow-up for research platforms

After browser-first validation, check whether the site also exposes a usable API or hosted MCP server. This matters when the user's real goal is to build a repeatable deep-research workflow in Hermes rather than only interact in the browser.

1. **Look for public API or MCP documentation after verifying the UI.**
   Search for:
   - OpenAPI/reference pages
   - `/api.json` or Swagger/Scalar docs
   - MCP docs pages and `.well-known/oauth-protected-resource`
   - OAuth/OpenID discovery documents for the auth server

2. **Separate public data endpoints from authenticated agent endpoints.**
   Verify with real requests when possible. Report separately:
   - endpoints that work anonymously (search, metadata, overview, similar papers, previews)
   - endpoints that return `401 Missing Authorization` or equivalent
   - MCP availability and transport (`url`/HTTP streamable server vs local stdio)

3. **For Hermes integration, prefer hosted MCP over scraping when it exists.**
   If the platform offers an MCP server and the user's goal is "Feynman-style deep research in Hermes," recommend Hermes-native MCP wiring first:

   ```yaml
   mcp_servers:
     <name>:
       url: "https://.../mcp/v1"
       auth: oauth
   ```

   Then use:
   - `hermes mcp add <name> --url <url> --auth oauth` or edit config manually
   - `hermes mcp login <name>` to force OAuth auth
   - `hermes mcp test <name>` to verify connectivity

4. **Headless/VPS OAuth is not a blocker.**
   Hermes supports paste-back OAuth for remote hosts. If the browser callback cannot reach the server, copy the final redirect URL (or just `?code=...&state=...`) back into the Hermes terminal.

5. **Setup pitfall: Hermes needs the MCP SDK installed.**
   If Hermes can list the `mcp` commands but runtime discovery fails because the Python package is missing, install `mcp` into the active Hermes environment first (for a Hermes-managed venv, `uv pip install mcp` is the safest default).

## Recommended reporting format

Use concise sections:

- **Verified public access**
- **Likely gated / paid**
- **API / MCP surface**
- **Not yet verified**
- **What I’d use in this environment**

## References

- `references/env-verification-pitfall.md` — safe pattern for checking Hermes credential-file configuration without being misled by redaction.
- `references/alphaxiv-notes.md` — example notes from validating a research site with mixed public and premium signals.
- `references/alphaxiv-mcp.md` — AlphaXiv-specific API/MCP notes: public endpoints, OAuth MCP URL, and Hermes integration path.
