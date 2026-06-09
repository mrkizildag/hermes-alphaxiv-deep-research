# Pivot from Authenticated Browser Session to API

Use this when a user is already logged into a web app in the browser and wants a structured dry run or export of the data they can see on the page (course list, task list, inbox, ticket queue, etc.).

## Reliable discovery pattern

1. Confirm the session is live by navigating to the relevant authenticated landing page.
2. In `browser_console`, inspect loaded resources to find candidate API endpoints:
   ```js
   performance.getEntriesByType('resource')
     .map(r => r.name)
     .filter(n => n.includes('/api/'))
   ```
3. Look for the two common shapes:
   - a **list/dashboard endpoint** that enumerates top-level containers (courses, projects, mailboxes, queues, etc.)
   - a **detail endpoint** that returns the nested items inside one container (assignments, tasks, threads, tickets)
4. If async `fetch(...)` evaluation is unreliable inside the browser tool, prefer a synchronous `XMLHttpRequest` for read-only `GET` probes — it serializes results predictably.

## Response shape — generic example

A typical list endpoint returns a top-level object with an array:

```jsonc
{
  "items": [
    { "container": { "id": 1, "title": "..." } },
    ...
  ]
}
```

A typical detail endpoint returns a single container with nested items:

```jsonc
{
  "container": {
    "id": 1,
    "title": "...",
    "items": [
      {
        "id": 42,
        "title": "...",
        "type": "...",
        "category": null,
        "shortName": "...",
        "dueDate": "...",
        "releaseDate": "...",
        "visibleToUsers": true
      }
    ]
  }
}
```

Useful fields to capture: `id`, `title`, `type`, `category`, `shortName`, `dueDate`, `releaseDate`, and any visibility/assignment flag.

Canonical item URLs are usually reconstructable from the container and item IDs:

```
https://<host>/<containerPath>/{containerId}/<itemPath>/{itemId}
```

## Filtering note

For category-style filtering (e.g. "real work" vs "tutorial/practice"), the API `category` field is frequently `null` or absent. Fall back to title heuristics — for example, excluding titles that contain words like `tutorial`, `practice`, or match short prefixes used by the app for practice items.

**Treat these heuristics as app-specific and user-reviewable, not universally correct labels.**

## Reporting pattern

For a dry run, return:

- active containers found
- total items discovered
- items excluded by heuristic, with the heuristic named
- resulting rows with stable fields like `container`, `title`, `type`, `dueDate`, plus the canonical `id` and `url`
- an explicit statement that no writes, syncs, or saved state were produced
