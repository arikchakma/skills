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

One to three bullets on what changed and why.

## Testing

What you ran, or what a reviewer should run.
```

Title: conventional-commit style, matching the branch's commits, 60 chars max (aim for 52). Use the user's arguments as the title when they gave one.

Never mention Claude, AI, or any co-author/generator attribution anywhere in the PR.

## Mermaid diagram

Only when the PR changes branching logic that exists in the diff. The test, applied before drawing anything: for every decision diamond, can you name the lines that implement it? If a diamond describes a judgement made in prose — a prompt, a doc, a style guide — then nothing branches and the diagram is decoration pretending to be logic.

No diagram for: prompt and copy changes, schema and type definitions, dependency bumps, config, pure styling, and refactors that move code without changing behaviour. A PR body with no diagram is a normal PR body.

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
