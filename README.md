# justincoulston-content

Public content for **justincoulston.com** — a diagram-native architecture publishing
platform. This repository holds the authored source (Markdown, draw.io diagrams, SVGs,
metadata) and is consumed by the private site repo as a git submodule at `content/`.

The site compiles this content at **build time** (`@jc/content`) into validated JSON +
pre-rendered HTML. Author here; the build does the rest. The browser never parses Markdown.

## Layout

```
articles/      # <slug>/article.md
diagrams/      # <id>/{diagram.svg, meta.json, diagram.drawio?}
notes/         # <slug>.md — short-form notes
office-hours/  # <slug>.md — office-hours sessions
resources/     # <slug>.md — curated resources
templates/     # reusable diagram/article templates
assets/        # shared images and files
```

The folder name (for articles/diagrams) or file name (for notes/resources/office-hours)
is the **id**, and must be `kebab-case`. Dates are ISO `YYYY-MM-DD` (write them unquoted;
the compiler normalizes them).

## Authoring schemas

The compiler validates every file with Zod and **fails the build** on invalid frontmatter
or a relationship that points at a non-existent id. The fields below are the contract.

### Article — `articles/<slug>/article.md`

```yaml
---
title: How I Stress-Test Software Architectures   # required
summary: One-sentence summary for cards and SEO.  # required
category: Distributed Systems                      # required (free-form)
publishedDate: 2026-05-20                           # required
updatedDate: 2026-06-02                             # optional
tags: [architecture, reliability]                  # optional
relatedArticles: [other-article-slug]              # optional; must exist
relatedDiagrams: [payment-authorization-flow]      # optional; must exist
keyTakeaways:                                        # optional
  - A short, standalone takeaway.
draft: false                                        # optional (default false)
---
```

Body is GitHub-flavored Markdown. Embed a diagram with a directive:

```markdown
:::diagram{id="payment-authorization-flow"}
:::
```

The id must match an existing diagram; the compiler fails the build otherwise. Headings
(`##`, `###`) become the table of contents; fenced code blocks are syntax-highlighted.

### Diagram — `diagrams/<id>/`

Each folder needs `diagram.svg` (rendered) and a metadata file — `meta.json` or
`metadata.json` — and may include `diagram.drawio` (downloadable source).

```json
{
  "title": "Payment authorization flow",
  "description": "What this diagram shows.",
  "category": "Payments",
  "publishedDate": "2026-05-18",
  "tags": ["payments", "integration"],
  "relatedArticles": ["stress-testing-software-architectures"],
  "relatedDiagrams": []
}
```

`category` must be one of: `Cloud`, `Payments`, `Multi-Tenant`, `Distributed Systems`,
`Security`, `Integration`.

### Note — `notes/<slug>.md`

```yaml
---
title: Coupling is a loan   # optional
publishedDate: 2026-06-06   # required
tags: [coupling]            # optional
---
```

Body is a short paragraph or two.

### Resource — `resources/<slug>.md`

```yaml
---
title: Designing Data-Intensive Applications   # required
url: https://dataintensive.net                  # required (valid URL)
description: Why it's worth your time.          # required
category: Books                                 # required
author: Martin Kleppmann                        # optional
tags: [data]                                    # optional
---
```

`category` must be one of: `Books`, `Tools`, `Templates`, `References`. The Markdown body
is an optional annotation.

### Office hours — `office-hours/<slug>.md`

```yaml
---
title: Architecture Review Office Hours   # required
summary: What the session is about.        # required
date: 2026-07-08                           # required
registrationUrl: https://lu.ma/...         # optional (valid URL)
---
```

## Diagram workflow

1. Create/edit the diagram in [diagrams.net](https://www.diagrams.net) and save the
   `.drawio` source.
2. Export an `.svg` (ensure a `viewBox` is present; avoid embedded fonts).
3. Commit both `diagram.drawio` and `diagram.svg` plus `meta.json`.
