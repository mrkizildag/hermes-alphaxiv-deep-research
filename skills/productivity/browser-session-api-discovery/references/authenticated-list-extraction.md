# Authenticated list extraction from a live browser session

Generic pattern for turning an authenticated app session into a structured list/dry-run payload via the app's own API, without relying on DOM scraping.

## Useful endpoint shapes

### 1. Active container dashboard

A list endpoint that enumerates the current user's active containers. Typical paths look like:

```
GET /api/<area>/<containers>/for-dashboard
GET /api/<area>/<containers>/active
GET /api/<area>/me/<containers>
```

Observed shape:

- top-level object with an `items` (or `courses`, `projects`, etc.) array
- each entry contains a nested container object
- useful minimum fields:
  - `container.id`
  - `container.title`

Common first-attempt mistake: assuming the response is a bare list when the actual payload is an object with a single array property inside it.

### 2. Container detail / items

A detail endpoint that returns one container plus its nested items:

```
GET /api/<area>/<containers>/{containerId}/for-dashboard
GET /api/<area>/<containers>/{containerId}/details
```

Observed useful fields:

- `container.id`
- `container.title`
- `container.items[]` (assignments, tasks, threads, tickets, etc.)

Observed item fields that mattered:

- `id`
- `title`
- `type` (e.g. `programming`, `quiz`, `file-upload`, `text`, `email`, `ticket`, ...)
- `dueDate`
- `releaseDate`
- `shortName`
- `visibleToUsers`
- `entityType`
- any team/assignee/computed-membership flag

Notably unreliable for classification: a `category` field is frequently absent or `null` in apps that conceptually have categories. Plan for fallback heuristics.

## Reliable browser-console extraction pattern

Async `fetch(...)` evaluation can produce tool-side 500s in some browser-tool implementations. A synchronous XHR snippet returns stable results.

Container list:

```js
(() => {
  const xhr = new XMLHttpRequest();
  xhr.open('GET', '/api/<area>/<containers>/for-dashboard', false);
  xhr.send(null);
  const items = JSON.parse(xhr.responseText).items;
  return items.map(x => ({ id: x.container.id, title: x.container.title }));
})()
```

Per-container detail:

```js
(() => {
  const xhr = new XMLHttpRequest();
  xhr.open('GET', `/api/<area>/<containers>/${containerId}/for-dashboard`, false);
  xhr.send(null);
  const c = JSON.parse(xhr.responseText).container;
  return c.items.map(e => ({
    container: c.title,
    id: e.id,
    title: e.title,
    type: e.type,
    dueDate: e.dueDate,
    url: `/<containerPath>/${c.id}/<itemPath>/${e.id}`,
  }));
})()
```

## Real-vs-practice filtering notes

Because the API often does not expose a clean classification flag, filtering requires heuristics plus UI confirmation.

### Heuristics that tend to work

Exclude titles matching practice/tutorial patterns such as:

- contains `tutorial`
- contains `practice`
- contains language-localized variants (`tutorium`, etc.)
- starts with a short practice-prefix used by the app (e.g. `P-...`, `T...`)

### Important caveat

Do **not** blindly exclude every title that contains the word `Tutorial`. Real items can include the word as part of the topic itself (for example, an item titled like `L06PE03 Tutorial Service` is about a "Tutorial Service" subsystem, not a tutorial-class exercise).

Always allow the user to review excluded items.

## "Upcoming only" refinement

Using the API alone is rarely enough to distinguish *future-but-completed* items from *future-and-incomplete* items.

Useful refinement pattern:

1. get live UTC time separately from a system clock tool
2. keep only future-due items
3. inspect the visible detail page for groupings or status text
4. exclude items visibly marked complete (`100%`, `Done`, equivalent localized strings)
5. exclude practice/tutorial entries from current sections

Common UI grouping labels to look for (localize as needed):

- `Due Soon` / `Bald Fällig`
- `Current` / `Aktuell`
- `Past` / `Vorangegangen`
- `No Date` / `Ohne Datum`

Common completion/status strings to look for:

- `100%`
- `Not started` / `Nicht angefangen` / `Nicht begonnen`
- `0% (preliminary)` / `0% (vorläufig)`
- `No graded result` / `Kein gewertetes Ergebnis`

## Dry-run row shape

Recommended normalized shape for a sync preview:

```jsonc
{
  "container": "...",
  "title": "...",
  "type": "programming|quiz|email|ticket|...",
  "dueDate": "2026-06-15T21:59:00Z",
  "itemId": 12345,
  "url": "/<containerPath>/{containerId}/<itemPath>/{itemId}"
}
```

## Main lesson

For any LMS-style or task-style app:

- enumerate active containers via a dashboard list endpoint
- enumerate items via a per-container detail endpoint
- treat real-vs-practice (or real-vs-tutorial) classification as partly heuristic unless a richer endpoint later proves otherwise
- validate "upcoming and incomplete" against the visible detail page, not against date alone
