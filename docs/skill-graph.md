# Skill Graph

## Included skills

### Core research skills
- `research/alphaxiv-deep-research`
- `research/authenticated-browser-research`
- `research/paper-relevance-catalog`

### Included helper skills
- `productivity/browser-session-api-discovery`
- `productivity/browser-auth-walkthroughs`

## Direct related-skill connections among included skills

- `alphaxiv-deep-research` → `authenticated-browser-research`
- `alphaxiv-deep-research` → `browser-session-api-discovery`
- `alphaxiv-deep-research` → `paper-relevance-catalog`
- `authenticated-browser-research` → `browser-auth-walkthroughs`
- `authenticated-browser-research` → `browser-session-api-discovery`
- `paper-relevance-catalog` → `alphaxiv-deep-research`

## Related skills referenced but not vendored

These are standard Hermes bundled skills, not custom local-only skills:
- `arxiv`
- `dogfood`
- `hermes-agent-skill-authoring`

## Linked files included

### `research/authenticated-browser-research`
- `references/alphaxiv-mcp.md`
- `references/alphaxiv-notes.md`
- `references/env-verification-pitfall.md`

### `productivity/browser-session-api-discovery`
- `references/alphaxiv-api-surface.md`
- `references/authenticated-list-extraction.md`

### `productivity/browser-auth-walkthroughs`
- `references/authenticated-api-pivot.md`
- `references/login-page-observation.md`

## Notes

- `alphaxiv-deep-research` has no linked reference files of its own, but it orchestrates the other included skills.
- `paper-relevance-catalog` is a pure output-format/triage companion skill and also has no linked reference files.
- The helper-skill references are written as generic browser-navigation lessons (list/detail endpoint discovery, login-page observation, authenticated API pivot) rather than as recipes against any specific site.
