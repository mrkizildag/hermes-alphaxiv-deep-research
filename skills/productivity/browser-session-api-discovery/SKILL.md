---
name: browser-session-api-discovery
description: Extract authenticated data from a live browser session by discovering and calling the web app's own API, then convert it into a dry-run/reporting payload.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [browser, api, authenticated-session, reverse-engineering, dry-run, reporting]
---

# Browser Session API Discovery

## Overview

Use this skill when the user is already logged into a web app in the browser and wants structured data out of it: lists, tasks, assignments, messages, invoices, tickets, or a sync dry-run. Prefer the app's authenticated API over scraping whenever possible.

The core move is:
1. open the live app in the browser,
2. discover real API endpoints from loaded resources or page behavior,
3. call those endpoints from the page context so the existing authenticated session is reused,
4. normalize the returned objects,
5. apply user-requested filtering,
6. present a dry-run table/payload before writing anywhere.

## When to Use

Use this when the user asks you to:
- pull data from a site where they are already signed in
- inspect a dashboard, portal, LMS, admin console, or SaaS app
- build or preview a sync into another system
- find undocumented API endpoints via browser/network behavior
- produce a "what would be pushed" dry run

Do not use this when:
- the site is public and `web_extract` is enough
- the task is only to log in interactively (use the browser auth walkthrough skill)
- the site blocks page-context API calls and plain scraping is clearly simpler

## Workflow

1. **Confirm the session is live by navigating to the target app.**
   - Start from the relevant dashboard or landing page.
   - Verify that authenticated content is visible before assuming the session exists.

2. **Discover endpoints from the app itself.**
   - First check whether the app exposes a public OpenAPI/Swagger/Scalar reference or machine-readable spec (`/api.json`, `/openapi.json`, `/docs`, dev/staging API hostnames, or `.well-known` metadata) before doing browser-only reverse engineering.
   - Inspect browser resources for `/api/` URLs.
   - Navigate into representative pages (course page, detail page, list page, etc.) to trigger richer API calls.
   - Prefer the most structured endpoint that returns the needed nested data.

3. **Probe anonymous vs authenticated surface area explicitly.**
   - Test a few likely read endpoints directly without credentials to learn what is already public.
   - Test likely account-scoped endpoints (`/assistant`, `/users`, `/me`, `/mcp`, etc.) and record whether they return `401/403` versus usable data.
   - If the service publishes OAuth/OpenID metadata, use it to determine the minimal credential shape you need before asking the user for secrets.

4. **Call the API from page context, reusing session auth when needed.**
   - In `browser_console`, prefer synchronous `XMLHttpRequest` snippets for one-shot extraction. They serialize reliably in tool evaluation and avoid async-return issues.
   - Parse the JSON in-page and return only the reduced fields you need, not the entire payload.

4. **Escalate endpoint discovery by page type.**
   - If the top-level dashboard response is too shallow, open a detail page and look for a course/project/entity-specific endpoint.
   - Many SPAs load more useful nested objects only on the detail route.

5. **Normalize to a durable row shape early.**
   - Example fields: source container, item title, category/type, due date, object ID, canonical URL, raw status flags.
   - Keep both human-readable labels and machine IDs when available.

6. **Filter conservatively and explain heuristics.**
   - If the API lacks an explicit category/status field, derive it from titles, labels, or page sections.
   - Say clearly when a filter is heuristic rather than first-class API data.

7. **For "upcoming only" requests, use live time and visible status where possible.**
   - Get current UTC time from the system with a tool.
   - Exclude past-due items by date.
   - Also exclude items visibly marked complete (e.g. `100%`, `completed`) on the app UI when the API response alone does not expose completion state cleanly.

8. **Return a dry-run artifact before any write path.**
   - Present a concise table and count.
   - If the user is building a sync, offer the exact target payload schema next.

## Preferred Patterns

### Discover API URLs from loaded resources
Use the page's own resource list to find candidate endpoints first.

### Use sync XHR for extraction
In browser-console contexts, a synchronous `XMLHttpRequest` often works more reliably than async `fetch(...)` evaluation when you need a direct returned value.

Pattern:
- open endpoint with `xhr.open('GET', url, false)`
- `xhr.send(null)`
- `JSON.parse(xhr.responseText)`
- map to a reduced list of rows

### Reduce in-page before returning
Tool contexts are fragile with giant payloads. Return only fields needed for the task.

## Pitfalls

1. **Skipping the documented API surface and going straight to DOM scraping.**
   If the app exposes OpenAPI docs, `.well-known` OAuth metadata, or a public machine-readable spec, map that first. It often reveals public endpoints, auth requirements, request shapes, and hidden capabilities much faster than browser-only inspection.

2. **Stopping at the first endpoint discovered.**
   Dashboard endpoints are often summaries only. Navigate deeper and inspect detail-page API calls.

3. **Returning full raw JSON blobs.**
   This wastes context and makes later filtering harder. Reduce immediately.

4. **Assuming category/status fields exist.**
   Many apps omit them or leave them null. Be ready to infer from naming or page grouping, and label the inference as heuristic.

5. **Using only due date to infer "upcoming and incomplete."**
   A future due date does not mean incomplete. Cross-check visible completion markers when available.

6. **Claiming exactness when the filter is title-based.**
   Be explicit about which exclusions came from heuristics like title prefixes or words such as Tutorial, Practice, Tutorium, etc.

7. **Writing to the destination system before showing a dry run.**
   For sync tasks, the first successful milestone is a verified preview.

## Verification Checklist

- [ ] Confirmed the app session was authenticated
- [ ] Identified at least one real API endpoint from page resources or route behavior
- [ ] Extracted structured data from page-context API calls
- [ ] Reduced results to the user-required schema
- [ ] Distinguished API facts from heuristic filtering
- [ ] Checked live time when the user asked for upcoming items
- [ ] Presented a dry-run table/count before any write action

## Support Files

- `references/authenticated-list-extraction.md` — generic LMS/task-app pattern: discovered list+detail endpoints, useful item fields, real-vs-practice filtering, and a normalized dry-run row shape.
- `references/alphaxiv-api-surface.md` — alphaXiv example: public OpenAPI discovery, anonymous-vs-authenticated endpoint probing, OAuth metadata checks, and MCP/assistant capability notes.
