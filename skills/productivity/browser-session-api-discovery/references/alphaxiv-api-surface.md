# alphaXiv API surface notes

Use these notes when a user asks what is possible through alphaXiv programmatically, especially before asking them to hand over credentials.

## Durable workflow

1. Check for a public OpenAPI reference before reverse-engineering browser traffic.
   - alphaXiv exposed an interactive API reference at `https://api-dev.alphaxiv.org/` and a machine-readable spec at `https://api-dev.alphaxiv.org/api.json`.
2. Separate **public unauthenticated endpoints** from **auth-gated agent endpoints** with direct probes.
3. Check OAuth metadata to learn what credential shape is needed.
   - `https://api.alphaxiv.org/.well-known/oauth-protected-resource`
   - `https://api.alphaxiv.org/.well-known/oauth-protected-resource/mcp/v1`
   - `https://clerk.alphaxiv.org/.well-known/openid-configuration`
4. Only then ask the user for the smallest credential that unlocks the needed path.

## Verified public endpoints

These worked without Authorization during the session:

- `GET /v2/papers/{upid}/metadata`
- `GET /search/v2/paper/fast?q=...&includePrivate=false`
- `GET /papers/v3/{id}/similar-papers`
- `GET /papers/v3/{paperVersion}/overview/{language}`
- `GET /papers/v3/{id}/preview`

This means alphaXiv can already support paper lookup, related-work discovery, overview retrieval, and metadata harvesting without user credentials.

## Verified auth-gated endpoints

These returned `401 {"error":{"message":"Missing Authorization"}}` without a bearer token:

- `GET /assistant/v2`
- `GET /users/v3`
- `GET /mcp/v1`

Treat the assistant/chat/user/MCP surface as authenticated by default unless a direct probe proves otherwise.

## Assistant API signals from the OpenAPI spec

Useful authenticated endpoints found in the public spec:

- `POST /assistant/v2/chat`
- `GET /assistant/v2`
- `GET /assistant/v2/{llmChat}/messages`
- `GET /assistant/v2/{llmChat}/context-window-usage`
- `GET /assistant/v2/messages/{messageId}/trace`
- `GET /assistant/v2/trace/{trace}`
- `POST /v1/assistant/upload-file`

Notable request fields on `POST /assistant/v2/chat` included:

- `message`
- `files`
- `llmChatId`
- `parentMessageId`
- `model`
- `thinking`
- `webSearch`
- `paperVersionId`
- `selectionPageRange`
- `assistantVariant`
- `plan`
- `signature`

This is a real research-assistant backend, not just a read-only paper API.

## MCP-specific notes

Docs page: `https://www.alphaxiv.org/docs/mcp`

Documented MCP endpoint:
- `https://api.alphaxiv.org/mcp/v1`

Documented tools:
- `discover_papers`
- `get_paper_content`
- `answer_pdf_queries`
- `read_files_from_github_repository`

The docs explicitly say browser-hosted MCP integrations are blocked by CORS and recommend a local bridge such as:
- `npx mcp-remote https://api.alphaxiv.org/mcp/v1`

## Credential guidance

Ask for the smallest reusable credential first:

- preferred: short-lived bearer access token
- header shape: `Authorization: Bearer <token>`

Do **not** ask for the user's password when a bearer token will do.

## General lesson

When the app exposes both a browser UI and a public API spec, do not start with DOM scraping alone. First map the documented API surface, test anonymous endpoints directly, and only then use the browser session for authenticated or hidden flows.