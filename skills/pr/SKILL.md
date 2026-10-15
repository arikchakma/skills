---
name: pr
description: create a pull request using the github cli
argument-hint: "Optional title or extra context for the PR"
---

Open a pull request for the current branch with `gh`.

## Preflight

- `gh auth status` — if it fails, tell the user to run `gh auth login` and stop.
- Confirm the current branch is not the default branch. If it is, stop and suggest calling the Skill tool with "create-branch" first.
- `git status --short` — if there are uncommitted changes, ask whether to commit them first (the `commit` skill) or leave them behind.
- Push with upstream tracking if needed: `git push -u origin HEAD`.
- `gh pr view --json url` — if a PR already exists, print its URL and stop.

## Build the description

Read the full range, not just the last commit:

```
git log --oneline origin/<default>..HEAD
git diff origin/<default>...HEAD
```

If `.github/PULL_REQUEST_TEMPLATE.md` (or `docs/`/`.github/PULL_REQUEST_TEMPLATE/`) exists, fill that template out and use nothing else.

Otherwise:

```markdown
## Summary

- Bullet one
- Bullet two

## Testing

What you ran, or what a reviewer should run.
```

The Summary section is those bullets and nothing else. No lead-in
sentence, no paragraphs between or after them, no "why this matters"
epilogue. If something doesn't fit in a bullet it belongs in the
commit message or a review comment, not the PR body.

One to three summary bullets, each 52-72 characters so a reviewer can
scan them in one pass. Write them the way you'd say it to a teammate:
plain english, active voice, what changed and why it matters. No
jargon dumps, no file paths, no "refactored the foo handler to
leverage the bar abstraction".

- Good: `Stop double-charging users who retry a failed payment`
- Bad: `Refactor PaymentService.retry() to dedupe idempotency keys`

Title: conventional-commit style, matching the branch's commits, 60 chars max (aim for 52). Use the user's arguments as the title when they gave one.

Never mention Claude, AI, or any co-author/generator attribution anywhere in the PR.

## Mermaid diagram

Default is no diagram. Almost every PR ships without one, including most PRs that add an `if`. A PR body with no diagram is the normal PR body.

Draw one only when all three hold:

1. The diff itself adds or changes control flow — not prose, copy, prompts, types, schema, config, dependency bumps, styling, or a refactor that moves code without changing behaviour.
2. There are three or more decision points that interact, and the outcome depends on which order they resolve in.
3. Every diamond maps to lines you can point at. A diamond describing a judgement made in prose — a prompt, a doc, a style guide — means nothing branches and the diagram is decoration pretending to be logic.

Two or three permission checks in a row are not a flowchart, they are a sentence. If you are weighing whether the diagram earns its place, it doesn't — ship without it.

When one does earn its place: `flowchart TD` following the path something real takes through the change — a request, a record, a job — with decision diamonds where the behaviour branches.

- Node labels are plain english for what happens (`Blocked · no email sent`), never file names, component names, endpoint paths or function names.
- No `subgraph` blocks, no database/service boxes, no architecture layers — it is a flow, not a system map.
- Under ~12 nodes; anything that does not fit goes in a one-line note under the diagram.

## Screenshots

Only for UI changes, and only when you have real screenshots (the user gave you paths, or you captured them with the browser tools / `run` skill). Never invent or promise images you don't have.

Attach with `--attach './file.png#alt text'` (works on both `gh pr create` and `gh pr edit`, up to 50 files). A `![alt](./file.png)` reference in the body is rewritten to point at the uploaded asset, so put the references where you want them:

- **New feature** — one screenshot of the new UI, referenced under a `## Screenshot` heading.
- **Changed existing UI** — before and after in a table:

```markdown
## Before / After

| Before | After |
| --- | --- |
| ![before](./before.png) | ![after](./after.png) |
```

Then `--attach ./before.png --attach ./after.png`.

## Create it

`gh pr create --title "..." --body "..."` — pass the body via `--body-file` with a heredoc to a temp file if it contains backticks.

Add `--draft` if the work is incomplete or the user asked. Do not add reviewers, labels, or assignees unless asked.

To amend a PR afterwards, use `gh pr edit` (see <https://cli.github.com/manual/gh_pr_edit>): `--title`, `--body`/`--body-file`, `--attach`, `--base`, `--add-label`/`--remove-label`, `--add-reviewer`, `--add-assignee` (`@me`, `@copilot`), `--milestone`. Without a body flag, `--attach` appends to the existing body instead of replacing it.

Print the PR URL when done.
