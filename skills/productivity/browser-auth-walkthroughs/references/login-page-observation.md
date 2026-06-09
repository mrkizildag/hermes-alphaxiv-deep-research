# Login Page Observation

Pattern for opening an unfamiliar sign-in page, reporting only what is actually visible, and then continuing the login flow when the user supplies credentials inline.

## Observed flow shape

A typical authenticated web app exposes:

1. A marketing/landing page with a visible `Log in` or `Sign in` entry point that routes to a path like `/sign-in`, `/login`, or `/auth`.
2. A sign-in page that may offer some subset of:
   - `Login or email` field
   - `Password` field
   - `Remember me`
   - `Forgot password?`
   - `Sign in` button
   - `Sign in with a passkey`
   - one or more SSO / federated buttons (Google, Microsoft, institutional SSO, etc.)
3. After successful authentication, the destination is some authenticated landing surface (dashboard, inbox, course overview, etc.) — not the sign-in page anymore.

## Durable lesson

Institutional or branding expectations ("there should be an SSO button here") are not enough.

**Report only the options that are actually visible on the page in the current session.** If an expected method is missing, say so explicitly rather than implying it exists.

## Browser-state lesson

When the user supplies credentials on a later turn, the prior browser session may already be gone or the element references may be stale.

The durable fix is:

- re-open the sign-in page
- take a fresh snapshot
- then type into newly resolved refs

The lesson is the refresh-and-resnapshot pattern, not the transient error itself.
