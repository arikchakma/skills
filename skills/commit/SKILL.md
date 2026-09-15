---
name: commit
description: stage and commit changes with conventional commits
argument-hint: "Optional scope or intent for the commit"
---

Stage and commit the current changes as one or more conventional commits.

## Steps

1. Run `git status --short` and `git diff` (plus `git diff --staged` if anything is already staged) to see everything that changed.
2. Run `git log --oneline -15` to match the repo's existing subject style — if it does not use conventional commits, follow what is there instead.
3. Group the changes by intent. One commit per logical change, not one commit per file. If everything is one concern, it is one commit.
4. For each group: `git add` exactly those paths (never `git add -A`), then commit.

## Message format

```
<type>(<optional scope>): <subject>

<optional body: why, not what>
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`.

- Subject: imperative mood, lowercase, no trailing period, 60 chars max (aim for 52).
- Body only when the diff does not explain itself. Skip it otherwise.
- Breaking change: `feat!:` or a `BREAKING CHANGE:` footer.

If the user passed arguments, treat them as the scope or the intent to describe.

## Rules

- Never commit secrets, `.env` files, credentials, or large build artifacts. Stop and say so if they appear in the diff.
- Do not amend, rebase, or force-push. Do not push unless the user asks.
- If a pre-commit hook fails, fix the cause and re-commit. Never use `--no-verify`.
- If there is nothing to commit, say so and stop.
