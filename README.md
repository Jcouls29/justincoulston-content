# justincoulston-content

Public content for **justincoulston.com** — a diagram-native architecture publishing
platform. This repository holds the authored source (Markdown, draw.io diagrams, SVGs,
metadata) and is consumed by the private site repo as a git submodule at `content/`.

The site compiles this content at **build time** (`@jc/content`) into validated JSON +
pre-rendered HTML. Author here; the build does the rest. The browser never parses Markdown.

## Layout

```
articles/      # <slug>/article.md (+ diagrams/, images/, metadata.json)
diagrams/      # <id>/{diagram.drawio, diagram.svg, meta.json}
notes/         # *.md — short-form notes
office-hours/  # *.md — office-hours session notes
resources/     # *.md (or resources.yaml) — curated resources
templates/     # reusable diagram/article templates
assets/        # shared images and files
```

> Authoring schemas and a full authoring guide land with Chunk 02 of the site build plan.
> Until then this is a skeleton: structure first, content next.
