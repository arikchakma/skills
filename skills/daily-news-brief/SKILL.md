---
name: daily-news-brief
description: summarize what is new on hacker news and github trending today
argument-hint: "Optional topic to filter for, or a section to run alone"
---

Build one brief of what shipped and what people argued about today. Two sources: Hacker News and GitHub Trending.

With an argument, keep only the items that match it. Without one, run every section.

## What to collect

| Section | Source | Items |
| --- | --- | --- |
| Front page | Hacker News front page | Top 10 |
| Show HN | Hacker News Show tab, last 48 hours | Top 10 |
| Trending | GitHub Trending, daily, all languages | Top 5 |
| TypeScript | GitHub Trending, daily, TypeScript | Top 5 |
| Rust | GitHub Trending, daily, Rust | Top 5 |

## Hacker News

Use the Algolia API. It needs no key, and it returns the comments in the same call.

```bash
# Front page, ranked
curl -s "https://hn.algolia.com/api/v1/search?tags=front_page&hitsPerPage=10"

# Show HN from the last 48 hours, ranked by points
T=$(( $(date +%s) - 172800 ))
curl -s "https://hn.algolia.com/api/v1/search?tags=show_hn&numericFilters=created_at_i%3E$T&hitsPerPage=10"

# The whole comment tree for one story
curl -s "https://hn.algolia.com/api/v1/items/<objectID>"
```

Each hit gives `title`, `url`, `objectID`, `points`, `num_comments`, `author`.

For each item:

1. Read the linked page or the repo to learn what the thing is. Do not guess from the title.
2. Read the comment tree. Take the top threads, not every reply.

Skip an item with fewer than about 10 comments. There is no discussion to report.

## GitHub Trending

There is no API. Read the three pages:

- `https://github.com/trending?since=daily`
- `https://github.com/trending?l=typescript&since=daily`
- `https://github.com/trending?l=rust&since=daily`

Each row gives the repo, the description, the language, the total stars, and the stars gained today. Open the README of a repo when the description does not say what it does.

A repo that appears on two lists goes in the brief once.

## Write each entry

Every entry in every section has the same three parts. Keep the shape identical, so the brief scans in one pass.

```markdown
### 1. [Title](https://example.com)
`412 points` · `180 comments` · [discussion](https://news.ycombinator.com/item?id=...)

**What it is.** Two sentences. What the project does, and who it is for. Name the
language or stack when it matters.

**What people say.** Three sentences. The strongest point in favor, the strongest
objection, and anything the author answered in the thread.
```

For GitHub, the second line carries `stars today` and the total instead, and **What people say** is dropped when there is no thread. Do not invent a reaction.

Rules for the summaries:

- Report the discussion, do not average it. A split thread reads as "half the thread says X, the rest says Y".
- Name the objection even when it is the minority view. That is the part worth reading.
- No hype words. "Fast" needs a number, or it is the author's claim and gets marked as one.
- Quote one short line when the thread turns on it.
- Skip an item that is a repost, a paywall with no substance, or a story with nothing new.

## Output

Publish an artifact when an Artifact tool is available. Load the `artifact-design` skill first, and build one page with:

- A title and the date.
- A section per table row above, in that order.
- Entries numbered, with the three parts in the same order every time.
- Nothing else. No charts, no filters, no search box.

When no Artifact tool exists, write `daily-news-brief-YYYY-MM-DD.md` in the current directory with the same structure, and print the path.

## Design

Build the page to the Vercel design system at <https://vercel.com/design.md>. Read that document before you write any CSS. It is the authority on type roles, spacing, color, and restraint. What follows is what it means for this brief.

- **Type.** Geist Sans for everything you read: headings, prose, labels, points, comment counts, star counts, ranks. Geist Mono only for identifiers, which here means repository names and file paths. Never set a whole sentence or a whole table in Mono.

```html
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Geist:wght@400..600&family=Geist+Mono:wght@400..600&display=swap" referrerpolicy="no-referrer">
```

- **Monochrome.** Near-black on white, near-white on black, with hairline rules. Add a color only when it encodes a state the reader must act on, and pair it with a cue that is not color.
- **No eyebrows.** Labels are sentence case at normal tracking. No uppercase, no letter-spacing, no pills, no badges.
- **No em dashes.** Use a period, a colon, or parentheses. The rule is explicit in the Vercel document, and it costs one pass over the copy.
- **One canvas.** No cards, no panels, no boxes around entries. Separate them with space and one hairline.
- **Measure.** Keep prose at 60 to 68 characters. Numbers get `font-variant-numeric: tabular-nums`.
- **Still.** No scroll reveals, no hover motion, no gradients, no shadows.

Keep the entry ranks. The Vercel document rejects decorative section numbers, and these are not decorative: they carry the Hacker News and GitHub Trending positions.

Do not add the Vercel wordmark, the triangle logo, or the authorship shell that the document specifies. Those mark a page as written by Vercel, and this brief is not. Take the design language and leave the branding alone.

Two limits when publishing as an artifact: the sandbox loads stylesheets only from `fonts.googleapis.com`, so `vercel-brand.css` cannot be linked and the tokens must be written into the page; and the artifact renders in the viewer's theme, so define both themes with tokens.

## Rules

- Every claim traces to a page you actually opened.
- Link every item. A brief with no links is not a brief.
- If a fetch fails, say which section is short and carry on. Do not fill the gap from memory.
- Today means today. Check the dates before you write.
