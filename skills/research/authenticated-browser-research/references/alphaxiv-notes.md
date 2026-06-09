# alphaXiv notes

Example of a research site that mixes public access with premium/account signals.

## Verified public surface

- Landing page loads without login.
- Public explore/discovery feed is visible.
- Individual paper pages under `/abs/...` can be opened.
- Public paper pages expose useful reading controls such as paper/blog/audio/info/download/share.
- `/overview/...` blog/summary pages can be opened publicly.
- `/audio/...` pages can be opened publicly and expose transcript text plus MP3 link.
- `Similar` results can be opened publicly from a paper/audio page.
- `Comments` UI can be opened publicly and exposes category toggles (`General`, `Research`, `Anonymous`) plus title/body fields.

## Research-agent surface vs actual execution

alphaXiv exposes a substantial assistant UI before login, including:

- `Assistant`
- `New Chat`
- `History`
- `Highlight & Ask`
- `Add Context`
- `Add GitHub link`
- `Literature reviews`
- `Community context`
- mode/style controls such as `Smart`, `Search`, `Style`

However, visible UI is **not** the same as usable agent execution.

### Verified gate

When a test question was entered into the assistant, alphaXiv showed an account gate with text equivalent to:

- `Join us`
- `Create an account to use our research agent and more!`
- `Continue with Google`
- `Sign up`
- `Log in`

This means the correct report is:

- **Assistant UI visible without login:** yes
- **Assistant execution verified without login:** no
- **Assistant execution triggers account gate:** yes

## Visible gating signals

- `Sign In` is prominently visible.
- `Upgrade to Pro` / `Pro` controls are visible.
- Sign-in page exposes both Google and email/password auth methods.

## Reporting pattern

For sites like this, report capabilities in tiers:

- **Verified public:** pages and controls you actually opened.
- **UI-visible but action-gated:** features whose panels render publicly but block on first meaningful action.
- **Likely gated:** features signaled by login/pro UI.
- **Not yet verified:** agentic workflows you did not run end-to-end.

## Environment-specific note

If the user says browser research should use a persistent Camofox session, prefer browser verification over plain search snippets. That session is the right source of truth for authenticated/dynamic behavior.
