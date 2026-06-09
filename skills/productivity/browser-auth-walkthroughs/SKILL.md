---
name: browser-auth-walkthroughs
description: Use when a user asks you to open a website, identify available sign-in methods, and walk them through authentication interactively in the browser.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [browser, authentication, login, walkthrough, credentials]
    related_skills: [dogfood, hermes-agent-skill-authoring]
---

# Browser Authentication Walkthroughs

## Overview

This skill covers interactive browser-based login assistance when the user wants you to inspect a sign-in page, report the available authentication methods, and then continue the login flow while they provide credentials inline.

The goal is to be concrete and stateful: open the site, enumerate the actual login options visible on the page, collect credentials only when needed, fill the form, submit it if authorized by the user, and verify success from the resulting authenticated page.

## When to Use

Use this when the user asks you to:
- open a website and inspect sign-in methods
- tell them whether the page offers SSO, direct login, passkey, magic link, etc.
- walk them through logging in step by step
- fill credentials they provide in chat
- confirm whether authentication succeeded and what page they landed on

Do not use this for:
- security policy advice unrelated to an actual browser flow
- account recovery flows that require email/SMS retrieval you cannot access
- bulk credential management or password storage

## Workflow

1. **Navigate to the target page first.**
   - Use the browser to open the site.
   - If the homepage is marketing content, click the visible sign-in/log-in entry point.

2. **Inspect and report the real options on the sign-in page.**
   - Enumerate only options actually visible: username/password, SSO button, passkey, forgot-password link, remember-me, etc.
   - Say explicitly when an expected option is *not* visible (e.g. no separate SSO button present).

3. **Prompt for credentials only after you know what fields are required.**
   - Ask for exactly the needed inputs (e.g. login/email and password).
   - Do not save credentials to memory or skills.

4. **Before typing on a later turn, refresh browser state.**
   - Browser sessions may not persist across turns or may lose element references.
   - Re-open the sign-in URL if needed and take a fresh snapshot before typing.
   - Never assume old element refs are still valid.

5. **Fill fields and submit in a minimal, deterministic sequence.**
   - Type username/email.
   - Type password.
   - Leave checkboxes alone unless the user asked to change them.
   - Submit via the primary sign-in button or Enter.

6. **Verify authentication from the destination page, not from the click itself.**
   - Check for authenticated indicators such as the username, account menu, dashboard, courses, inbox, or a logout button.
   - If the URL stays on the sign-in page, inspect for inline error messages before claiming failure or success.

7. **When the real task starts after login, pivot from UI to authenticated data access.**
   - If the user wants course lists, assignments, dashboards, or other structured account data, inspect browser resources / network activity for the app's REST endpoints before scraping the DOM.
   - In browser_console, first read `performance.getEntriesByType('resource')` and filter for `/api/` to discover likely endpoints already used by the app.
   - Prefer calling the same authenticated endpoints the page uses. This is usually more complete and less brittle than scraping visible cards or tables.
   - If `fetch(...).then(...)` or async IIFEs are flaky in the browser evaluator, retry with a synchronous `XMLHttpRequest` probe for read-only GET requests and return a compact JSON summary rather than the full payload.
   - For per-course dashboards, fetch the course list first, then iterate course detail endpoints to collect exercises or assignments; capture IDs, titles, types, due dates, and canonical URLs from the API response.

8. **Summarize the result succinctly.**
   - State whether login succeeded.
   - State which method was used.
   - Mention the current landing page and any immediately visible next actions.
   - If the user asked for a dry run, explicitly state that no write-back or persistence actions were performed.

## Reporting Template

Use this structure when responding:

1. **Login options visible**
   - direct login with username/password
   - SSO / passkey / other alternate methods
   - notable auxiliary controls (forgot password, remember me)

2. **What I need from you**
   - request only the missing credentials or confirmation

3. **After submission**
   - whether login succeeded
   - what page is open now
   - a short list of relevant next actions the user can ask for

## Common Pitfalls

1. **Typing into stale browser refs after the user replies on a later turn.**
   Fix: re-navigate or re-snapshot before typing credentials.

2. **Claiming login succeeded immediately after clicking submit.**
   Fix: verify the post-login page for authenticated UI or inspect for errors.

3. **Inferring SSO availability from branding or institutional context.**
   Fix: report only the options actually visible on the sign-in page.

4. **Over-asking before inspecting the page.**
   Fix: first determine whether the page wants email/password, SSO only, passkey, or some other method.

5. **Persisting sensitive credentials.**
   Fix: use them only for the live browser step and do not store them in memory or skills.

## Verification Checklist

- [ ] Opened the real sign-in page, not just the marketing homepage
- [ ] Enumerated only the auth methods actually visible
- [ ] Requested only the credentials actually needed
- [ ] Re-snapshotted or re-opened the page before typing on a later turn
- [ ] Verified success from authenticated UI or captured the on-page error state
- [ ] Reported the landing page and next useful actions

## References

- `references/login-page-observation.md` — generic pattern for opening an unfamiliar sign-in page, reporting only what's actually visible, and surviving stale browser state across turns.
- `references/authenticated-api-pivot.md` — how to pivot from an authenticated browser session into REST endpoint discovery and a dry-run extraction of container/item data.
