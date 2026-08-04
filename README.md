# Problem Catalog Viewer

This viewer is designed for JSON records describing mathematical, algorithmic, or scientific open problems. It supports both the original **LITchikin legacy format** and the newer extended format.

Open `viewer.html` in a browser, then use **Upload JSON Files** to load either a repository catalog or a local JSON file. Uploads are read and parsed in the browser.

## Repository layout

```text
.
├── viewer.html
├── gener_ds.html
├── open_problem_record.schema.json
└── data/
    └── SimonsF25ComplexityLinAlg.json
```

All catalog JSON files belong in `data/`. The viewer remains at the repository root. The repository's default catalog is `data/SimonsF25ComplexityLinAlg.json`.

## Accepted top-level shapes

The viewer accepts all of the following:

### 1\. One problem object

```json
{
  "problem": "Problem statement",
  "wordDescription": "Expository explanation",
  "mathematicalConditions": "Formal restatement",
  "literature": \\\\\\\\\\\\\\\[]
}
```

### 2\. An array of problem objects

```json
\\\\\\\\\\\\\\\[
  {
    "problem": "First problem",
    "wordDescription": "...",
    "mathematicalConditions": "...",
    "literature": \\\\\\\\\\\\\\\[]
  },
  {
    "problem": "Second problem",
    "wordDescription": "...",
    "mathematicalConditions": "...",
    "literature": \\\\\\\\\\\\\\\[]
  }
]
```

### 3\. A combined catalog

```json
{
  "schemaVersion": "1.2",
  "title": "My problem collection",
  "problems": \\\\\\\\\\\\\\\[
    {
      "problem": "Problem statement",
      "wordDescription": "...",
      "mathematicalConditions": "...",
      "literature": \\\\\\\\\\\\\\\[]
    }
  ]
}
```

The wrapper array may also be named `items`, `records`, or `data`.

## Legacy LITchikin format

This is the format used by the earlier JSON files in the LITchikin repository.

```json
{
  "problem": "What is the complexity of ...?",
  "wordDescription": "A plain-language explanation of the question and its importance.",
  "mathematicalConditions": "A formal statement using LaTeX where useful.",
  "literature": \\\\\\\\\\\\\\\[
    {
      "title": "Author (Year) - Paper title",
      "year": 2025,
      "description": "Why the paper is relevant.",
      "link": "https://arxiv.org/abs/..."
    }
  ],
  "generatedAt": "2026-08-03T14:00:00Z"
}
```

The viewer automatically supplies display fallbacks for metadata absent from a legacy record:

* `sourceStatement` defaults to `problem`.
* `status` is shown as `unspecified`.
* A problem number is inferred from filenames such as `simonsf25-2-10.json`.
* A verification note explains that source/status metadata was not present.
* `generatedAt` is displayed when available.

The viewer also repairs a common legacy issue: **literal line breaks inside quoted JSON strings**. New files should nevertheless be emitted as valid JSON.

## Extended format

The canonical extended record is:

```json
{
  "sourceStatement": "Verbatim or lightly normalized source text",
  "problem": "Cleaned statement without changing the intended meaning",
  "wordDescription": "Expository explanation and importance",
  "mathematicalConditions": "Formal restatement",
  "comments": [],
  "literature": \\\\\\\\\\\\\\\[
    {
      "title": "Author (Year) - Paper title",
      "year": 2025,
      "description": "How this work mentions, attempts, resolves part of, or supplies tools for the problem.",
      "link": "https://..."
    }
  ],
  "source": {
    "title": "Source document title",
    "authors": \\\\\\\\\\\\\\\["First Author", "Second Author"],
    "url": "https://...",
    "page": 17,
    "problemNumber": "2.10"
  },
  "status": "open",
  "lastVerified": "2026-08-03",
  "tags": \\\\\\\\\\\\\\\["linear systems", "bit complexity"],
  "verificationNotes": "What was checked, what remains uncertain, and whether the literature list is exhaustive.",
  "generatedAt": "2026-08-03T14:00:00-04:00"
}
```

### Field definitions

|Field|Type|Meaning|
|-|-|-|
|`problem`|string|The cleaned problem statement. This is the only strongly recommended field.|
|`wordDescription`|string|Explanation, motivation, and importance.|
|`mathematicalConditions`|string|Formal assumptions, quantifiers, success criteria, and meaningful partial progress.|
|`comments`|array|Chronological user comments associated with this problem. Use an empty array when there are no comments.|
|`literature`|array|Relevant papers, books, posts, or source mentions.|
|`sourceStatement`|string|Verbatim or minimally normalized wording from the source.|
|`source`|object|Source title, authors, URL, page, and problem number.|
|`status`|string|Usually `open`, `partially resolved`, `resolved in stated regime`, or `uncertain`.|
|`lastVerified`|string|ISO date on which the status and source were checked.|
|`tags`|array of strings|Search/filter labels.|
|`verificationNotes`|string|Scope, ambiguities, source errors, and verification limitations.|
|`generatedAt`|string|ISO timestamp for record generation.|

### Comment entries

Comments are stored directly in each problem record as a flat chronological array:

```json
{
  "id": "comment-01HXYZ",
  "author": {
    "id": "user-123",
    "displayName": "Ada"
  },
  "body": "A Markdown or LaTeX-enabled comment.",
  "createdAt": "2026-08-04T16:00:00Z",
  "updatedAt": null
}
```

`id`, `author`, `body`, and `createdAt` are required for non-empty comment entries. `author` may also be a string for legacy or imported data. Additional comment fields are allowed for forward compatibility.

### Literature entries

Canonical literature entries use:

```json
{
  "title": "Paper or source title",
  "year": 2025,
  "description": "Specific relevance to this problem",
  "link": "https://..."
}
```

For compatibility, the viewer also accepts:

* A string instead of an object.
* `name`, `citation`, or `paper` instead of `title`.
* `summary` or `notes` instead of `description`.
* `url`, `href`, or `doi` instead of `link`.
* `references`, `relatedWork`, `relatedLiterature`, or `bibliography` instead of `literature`.

## Accepted field aliases

The viewer normalizes these aliases:

|Canonical field|Accepted aliases|
|-|-|
|`problem`|`question`, `title`, `statement`|
|`wordDescription`|`description`, `explanation`, `exposition`, `importance`|
|`mathematicalConditions`|`conditions`, `formalStatement`, `formalRestatement`, `mathDescription`|
|`sourceStatement`|`sourceText`, `verbatimSourceText`, `originalStatement`|
|`literature`|`references`, `relatedWork`, `relatedLiterature`, `bibliography`|
|`tags`|a string or an array|
|`source.url`|top-level `sourceUrl`|
|`source.page`|top-level `sourcePage`|
|`source.problemNumber`|top-level `problemNumber` or `number`|

## LaTeX and JSON escaping

MathJax renders `$...$`, `$$...$$`, `\\\\\\\\\\\\\\\\(...\\\\\\\\\\\\\\\\)`, and `\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\[...\\\\\\\\\\\\\\\\]`.

Because JSON treats backslash as an escape character, LaTeX commands require doubled backslashes:

```json
{
  "problem": "Estimate $\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\operatorname{vol}(K)$ in $\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\mathbb{R}^n$."
}
```

Use `\\\\\\\\\\\\\\\\n` for a line break inside a JSON string. Do not place an unescaped literal newline between the opening and closing quotation marks.

Correct:

```json
{
  "wordDescription": "First paragraph.\\\\\\\\\\\\\\\\nSecond paragraph."
}
```

## Prompt for an LLM

The following prompt can be copied into an LLM:

```text
Return only valid JSON, with no Markdown fence and no commentary.

Create one open-problem record with this canonical schema:
{
  "sourceStatement": "verbatim or minimally normalized source wording",
  "problem": "cleaned statement without changing meaning",
  "wordDescription": "clear explanation of the problem and why it matters",
  "mathematicalConditions": "formal restatement with assumptions, quantifiers, target guarantees, and meaningful partial progress",
  "comments": [],
  "literature": \\\\\\\\\\\\\\\[
    {
      "title": "Author (Year) - Title",
      "year": 2025,
      "description": "specific relevance to the problem",
      "link": "https://..."
    }
  ],
  "source": {
    "title": "source title",
    "authors": \\\\\\\\\\\\\\\["author names"],
    "url": "source URL",
    "page": 1,
    "problemNumber": "1.1"
  },
  "status": "open",
  "lastVerified": "YYYY-MM-DD",
  "tags": \\\\\\\\\\\\\\\["tag one", "tag two"],
  "verificationNotes": "what was verified and any limitations",
  "generatedAt": "ISO-8601 timestamp"
}

Requirements:
- Preserve the source's terminology and meaning.
- Distinguish source-derived facts from inference.
- Do not claim the literature list is exhaustive unless it was systematically verified.
- Use valid JSON double quotes.
- Escape every LaTeX backslash as a double backslash.
- Encode line breaks inside strings as \\\\\\\\\\\\\\\\n, not as literal unescaped newlines.
- Use null or omit a field when the source does not supply it; do not invent metadata.
```
