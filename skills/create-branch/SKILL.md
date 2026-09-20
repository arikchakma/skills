---
name: create-branch
description: create a branch off the default branch using conventional naming
argument-hint: "What the branch is for"
---

Create a new branch off the freshly-fetched default branch.

## Steps

1. Find the default branch: `git symbolic-ref --short refs/remotes/origin/HEAD` (fall back to `main`, then `master`).
2. Check for uncommitted work with `git status --short`. If there is any, ask the user whether to bring it along (`git switch -c` carries it) or stash it first. Do not discard anything.
3. `git fetch origin <default>`, then `git switch -c <name> --no-track origin/<default>` so the branch is cut from a fresh copy without inheriting the default branch as its upstream.
4. Report the branch name and what it was cut from.

## Naming

```
<type>/<short-kebab-description>
```

Types match conventional commit types: `feat`, `fix`, `docs`, `refactor`, `perf`, `test`, `chore`.

- Three to five words, kebab-case, lowercase.
- Lead with the ticket ID when the repo's history shows that convention: `feat/PROJ-123-oauth-login`.
- Check `git branch -a --format='%(refname:short)' | head -30` first and match whatever pattern is already in use — the repo's convention wins over this one.

Derive the name from the user's arguments. If they gave none, derive it from the uncommitted changes or the conversation, and ask only when there is nothing to go on.

## Rules

- Never commit or push — creating the branch is the whole job.
- `--no-track` is not optional. Without it git sets the new branch's upstream to `origin/<default>`, and the next push — from the CLI or a GUI's push button — lands the branch's commits directly on the default branch. If the output says `set up to track 'origin/<default>'`, fix it with `git branch --unset-upstream` before going further; the upstream gets set to the branch's own name on first push (`git push -u origin HEAD`).
- Never reset, rewrite, stash away, or discard existing work to make the branch.
- If a branch with that name already exists, switch to it and say so instead of failing.
