# LITchikin problem catalogs

LITchikin stores open-problem code and data in one repository. All repository
catalogs live in `data/` and use the strict `1.4` format defined by
`open_problem_record.schema.json`.

## Repository layout

```text
.
├── data/
│   └── SimonsF25ComplexityLinAlg.json
├── gener_ds.html
├── open_problem_record.schema.json
└── viewer.html
```

`gener_ds.html` creates a one-problem catalog in the browser. `viewer.html`
temporarily reads one or more local catalog files in the browser. Neither page
uploads a selected JSON file to a server.

## Strict catalog format (`1.4`)

Only the combined catalog shape is canonical:

```json
{
  "schemaVersion": "1.4",
  "catalogId": "example-catalog",
  "generatedAt": "2026-08-06T01:00:00.000Z",
  "contentUpdatedAt": "2026-08-06T01:00:00.000Z",
  "problemCount": 1,
  "problems": [
    {
      "problem": "State the open problem.",
      "wordDescription": "Explain its motivation.",
      "mathematicalConditions": "Give its formal conditions.",
      "comments": [],
      "literature": [],
      "generatedAt": "2026-08-06T01:00:00.000Z",
      "contentUpdatedAt": "2026-08-06T01:00:00.000Z",
      "recordId": "lit:example-catalog:1"
    }
  ]
}
```

The viewer rejects standalone problem objects, top-level arrays, older schema
versions, missing stable IDs, missing content timestamps, and non-UTC content
timestamps.

## Stable IDs

Every catalog has a lowercase kebab-case `catalogId`, for example
`simons-f25-complexity-linalg`. Every problem has a stable `recordId`:

```text
lit:<catalogId>:<record-suffix>
```

Use a source problem number as the suffix when one exists (`2.3`, `5.meta`).
Otherwise choose a short stable slug. Never derive an existing record ID again
after its title or wording changes.

## Time fields

- `generatedAt` records creation time and is optional historical metadata.
- `contentUpdatedAt` is required on both the catalog and every problem.
- Stored timestamps use UTC ISO 8601 and end in `Z`.
- A problem's `contentUpdatedAt` changes only when its non-comment content
  changes.
- The catalog timestamp changes when catalog metadata changes or a problem is
  added, removed, or has non-comment content changed.
- Adding or editing a comment does not change a content timestamp.

There is no fallback from `contentUpdatedAt` to `generatedAt`. Direct data
editors must update the explicit content timestamps. The Fizzy admin commit flow
compares the working catalog with Git `HEAD` and maintains them automatically.

## Comments

Comments are stored directly in each problem's chronological `comments` array:

```json
{
  "id": "comment-01HXYZ",
  "author": {
    "id": "user-123",
    "displayName": "Ada"
  },
  "body": "A Markdown or LaTeX-enabled comment.",
  "createdAt": "2026-08-06T01:00:00.000Z",
  "updatedAt": null
}
```

`id`, `author`, `body`, and `createdAt` are required for non-empty entries.
`author` may be a string for imported historical data.

## Generator

In `gener_ds.html`:

1. Enter a problem description and generate/edit its content.
2. Enter a lowercase kebab-case catalog ID.
3. Enter the stable record suffix.
4. Download the resulting strict `1.4` catalog JSON.

The generator writes the same UTC timestamp into `generatedAt` and
`contentUpdatedAt` for a newly created catalog and problem.

## Viewer

Open `viewer.html` and select one or more `1.4` catalog JSON files. All parsing
is local to that browser tab. The viewer validates the schema version, catalog
ID, catalog timestamp, record IDs, record timestamps, and duplicate IDs before
showing any problems from a file.

MathJax renders `$...$`, `$$...$$`, `\\(...\\)`, and `\\[...\\]`. Because JSON
uses backslash escapes, LaTeX backslashes must be doubled inside JSON strings.
