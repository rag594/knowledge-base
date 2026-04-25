# LLM Wiki — Schema & Operating Instructions

**Domain:** Blogs, articles, and interesting things found on the internet.

This is a persistent knowledge base maintained by Claude. You (Claude) own the `wiki/` layer entirely. You read from `raw/`; you never modify it. The human curates sources and asks questions; you do all the filing, cross-referencing, and maintenance.

---

## Directory layout

```
LLMWiki/
├── CLAUDE.md           ← this file — your operating instructions
├── index.md            ← content catalog (update on every ingest)
├── log.md              ← append-only chronological record (append only)
├── raw/                ← source documents — NEVER modify
│   └── assets/         ← locally downloaded images/attachments
└── wiki/               ← everything you write
    ├── concepts/        ← idea/topic/theme pages
    ├── entities/        ← author/publication/person/org pages
    └── sources/         ← one summary page per ingested article or blog post
```

---

## Tag vocabulary

Use these tags consistently across all pages. Add new ones to this list when a clear gap exists.

**Topic tags:** `ai`, `ml`, `software`, `systems`, `startups`, `business`, `science`, `philosophy`, `design`, `economics`, `psychology`, `culture`, `writing`, `productivity`, `health`, `politics`, `math`, `security`, `hardware`, `web`

**Format tags:** `blog-post`, `essay`, `research`, `newsletter`, `tutorial`, `opinion`, `interview`, `thread`

**Quality flags:** `must-read`, `worth-revisiting`, `disagree`, `speculative`

---

## Page formats

### Source summary (`wiki/sources/<slug>.md`)

```markdown
---
title: <article title>
author: <author name or handle>
publication: <blog name, newsletter, or domain>
url: <original URL>
date_published: <YYYY-MM-DD or approximate year, leave blank if unknown>
date_read: <YYYY-MM-DD>
tags: [<topic-tag>, <format-tag>, <quality-flag if applicable>]
---

## Summary

2–4 paragraphs. What does the piece argue, show, or explore? What is the central idea? What makes it worth reading?

## Key points

- Bullet list of the most important takeaways, claims, or observations.

## What's interesting

1–2 sentences on what stood out — a surprising claim, an elegant argument, a useful frame, or something worth thinking about further.

## Authors & publications

Links to entity pages: [[Author Name]], [[Publication Name]]

## Concepts

Links to concept pages that this article touches: [[Concept Name]]

## Connections

How does this piece relate to other things already in the wiki? What does it confirm, challenge, or extend?
```

### Author page (`wiki/entities/<slug>.md`)

```markdown
---
title: <author name or handle>
type: entity
entity_type: author
tags: []
---

## About

1–3 sentences: who are they, what do they write about, where do they publish?

## Articles in this wiki

| Title | Slug | Date read |
|-------|------|-----------|
| ...   | ...  | ...       |

## Key ideas associated with this author

Synthesis of recurring themes or arguments across their work: [[Concept]]

## Related

[[Publication Name]], [[Other Author]]
```

### Publication page (`wiki/entities/<slug>.md`)

```markdown
---
title: <blog / newsletter / site name>
type: entity
entity_type: publication
url: <homepage URL>
tags: []
---

## About

What is this publication? What does it cover? Who runs it?

## Articles in this wiki

| Title | Slug | Date read |
|-------|------|-----------|
| ...   | ...  | ...       |

## Related authors

[[Author Name]]
```

### Concept page (`wiki/concepts/<slug>.md`)

```markdown
---
title: <concept name>
type: concept
tags: []
---

## Definition

Clear 1–3 sentence definition or framing.

## Discussion

Synthesis across sources — how is this concept treated, argued about, or developed across the articles you've read?

## Key sources

| Article | What it contributes |
|---------|---------------------|
| [[source-slug]] | one-line note |

## Related concepts

[[Concept A]], [[Concept B]]
```

### Analysis / query output page (`wiki/<slug>.md`)

For non-trivial answers worth keeping:

```markdown
---
title: <question or topic>
type: analysis
date: <YYYY-MM-DD>
sources: [<slug>, <slug>]
tags: []
---

Content of the analysis.
```

---

## File naming convention

- Lowercase, hyphen-separated: `paul-graham.md`, `attention-is-all-you-need.md`, `stratechery.md`.
- Source slugs: derive from the article title — strip punctuation, lowercase, replace spaces with hyphens, keep it short (≤ 6 words).
- No special characters in filenames.

---

## Linking convention

- Always use `[[slug|Display Text]]` format for all wiki links — e.g. `[[thundering-herd|Thundering Herd]]`, `[[bohan-zhang|Bohan Zhang]]`. **Never use `[[Title Case With Spaces]]`** — Quartz converts spaces to hyphens but does not lowercase, producing broken links (`../Thundering-Herd` instead of `../thundering-herd`).
- Every author name, publication name, and concept term that has (or should have) a page must be linked on every appearance.
- When you create a new page, add backlinks from relevant existing pages.

---

## Operations

### 1. Ingest

When the human drops an article into `raw/` or pastes a URL/text and says to process it:

1. **Read** the source.
2. **Discuss** briefly — ask if there's anything specific to emphasize, or proceed if the human says to just process it.
3. **Write** a source summary page in `wiki/sources/`.
4. **Update or create** an author entity page. If the author has prior articles in the wiki, add this one to their table and note any new themes.
5. **Update or create** a publication entity page.
6. **Update or create** concept pages for every significant idea or theme.
7. **Add cross-links** — scan existing wiki pages for mentions of this author, publication, or concept and link the new source from them.
8. **Update `index.md`** — add the new source row and any new wiki pages.
9. **Append to `log.md`** — one entry.

A typical ingest touches 5–10 pages. Cross-links are where the value compounds — be thorough.

### 2. Query

When the human asks a question:

1. Read `index.md` to identify relevant pages.
2. Read the relevant wiki pages.
3. Synthesize an answer with inline citations (`[[source-slug]]`).
4. Offer to persist the answer as an analysis page if it's non-trivial or likely to be useful later.

Useful output formats:
- **Narrative:** a flowing answer with citations.
- **Comparison table:** useful for "how does X compare to Y across articles."
- **Reading list:** curated list of sources from the wiki on a topic, with one-line annotations.
- **Marp slide deck:** add `marp: true` to frontmatter and use `---` as slide separators.

### 3. Lint

When the human asks for a health check:

1. Find concepts or authors mentioned in multiple source pages but lacking their own page.
2. Find orphan pages with no inbound links.
3. Find factual contradictions or conflicting claims between pages.
4. Find concept pages that have only one source — flag for enrichment.
5. Suggest articles or topics to look for based on gaps in coverage.
6. Report as a bulleted list. Fix what you can in place; flag what needs human judgment.

---

## index.md structure

```markdown
# Wiki Index

_Last updated: YYYY-MM-DD_

## Sources
| Title | Author | Slug | Date read | Tags |
|-------|--------|------|-----------|------|

## Entities — Authors
| Name | Slug | Articles in wiki |
|------|------|-----------------|

## Entities — Publications
| Name | Slug | Articles in wiki |
|------|------|-----------------|

## Concepts
| Name | Slug | Source count |
|------|------|-------------|

## Analyses
| Title | Date | Slug |
|-------|------|------|
```

---

## log.md structure

Append-only. One `##` heading per entry, grep-friendly prefix.

```
## [YYYY-MM-DD] <operation> | <title>

One-sentence note.
```

Operations: `ingest`, `query`, `lint`, `update`.

---

## Style rules

- **Synthesize, don't transcribe.** The summary should say what the piece *means*, not just what it *says*.
- **One claim, one source.** When adding a fact to an entity or concept page, note which article it comes from.
- **Flag contradictions.** If a new article contradicts an existing claim, note both positions and which is more recent.
- **No duplication.** Source summaries hold detail; concept pages hold synthesis. Don't paste the same paragraph in both.
- **Prefer updating over creating.** Expand an existing concept page before creating a new near-duplicate.
- **Link generously.** Every entity and concept with a page gets linked every time it appears.

---

## Publishing

The wiki is deployed to `wiki.raghavrastogi.in` via Quartz + GitHub Pages. After each ingest session, remind the human to publish:

```
git add -A && git commit -m "ingest: <article title>" && git push
```

Quartz builds from the `wiki/` directory (`npx quartz build --directory wiki`). The `raw/` directory is in git but not published. The GitHub Actions workflow is at `.github/workflows/deploy.yml`.

---

## Customization

This schema is a living document. Update it when you and the human agree on a change. Note the *why* when changing a convention so future sessions understand the reasoning.
