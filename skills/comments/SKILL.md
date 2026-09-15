---
name: comments
description: write comments that earn their place, and delete the ones that do not
argument-hint: "Optional path or scope to clean"
---

Clean the comments in the code under discussion, and write new ones the same way.

With arguments, work on those paths. Without them, work on the current diff (`git diff` plus `git diff --staged`), or on the file the user is asking about.

## The test

A comment earns its place when it says something the code cannot. Everything else is noise a reader must skip.

Before you write one, try to delete the need for it:

1. Rename the variable or function. `elapsedMs` needs no comment. `t` does.
2. Name the constant instead of leaving it magic. `MAX_RETRIES = 3` beats `3` plus a comment.

Write the comment only when neither works. Do not extract a function to get rid of a comment. A one-caller function invented for that reason is worse than the comment it replaced: the reader now jumps to another place to learn the same thing.

## What to keep and write

- **Why this exists.** The constraint, the bug, the business rule that forces the code into this shape.
- **Why not the obvious thing.** The approach you rejected and what broke when you tried it.
- **A link.** A ticket, an RFC, a vendor bug. One URL is worth a paragraph.
- **A warning.** An invariant a future editor can break without noticing.
- **A known ceiling.** A deliberate simplification and when to upgrade it.
- **Sources if the comments needs it.** An issue link or a reference to the code, use the `@see` tag. (Add only if the comment actually needs it.)

```js
// Stripe sends this webhook twice for the same charge. Key on event.id,
// not charge.id, or the customer gets billed again.
// 
// @see https://github.com/stripe/stripe-node/issues/
```

## What to delete

- Restatements of the line below. `// increment the counter` above `counter++`.
- Doc blocks that repeat the signature and add nothing. `@param userId The user ID.`
- Changelog notes in the source. `// added by Sam 2024-03-11`. Git has this too.
- Section banners. `// ===== HELPERS =====`.
- Narration of a refactor. `// now using the new API`. Now is not a fixed point in time.
- Praise or apology. `// clever trick`, `// sorry about this`.

Leave a `TODO` or `FIXME` that names a ticket or issue. Leave a linter or type-checker directive, such as `eslint-disable-next-line`. Leave a license header.

## How to word it

Follow [ASD-STE100](https://asd-ste100.org/) and the [Google developer documentation style guide](https://developers.google.com/style). For longer prose, call the Skill tool with "simple-english".

- Short declarative sentences. Twenty words is the ceiling.
- Present tense, active voice, and name the actor. "Postgres locks the table", not "the table is locked".
- One idea per sentence.
- Common word over jargon. Use "use" not "utilize", "start" not "initiate".
- One item, one name. Do not call it "config" here and "settings" two lines down.
- `can`, `will`, `must`. Avoid "should", "may", "might".
- No hedging and no filler. Say what is true, not that it is important to note.

**Before:** `// It's important to note that this function will basically handle the processing of the user data that gets passed in.`
**After:** `// Trims whitespace before validation. The signup form sends padded values.`

## JSX

Do not put comments inside JSX or inside JSX prop expressions. They render as noise in the tree and break prop formatting.

```jsx
// Bad
<Button
  {/* disabled until the plan loads */}
  disabled={!plan}
/>

// Good: put it above the element, or above the component.
{/* Plan loads async. Guard the action until it arrives. */}
<Button disabled={!plan} />
```

## Rules

- Change comments only. Do not touch the logic in a cleaning pass. No extracting, no splitting, no reshaping.
- Do not add comments to code that reads fine without them. Most code does.
- Match the file. If it uses JSDoc on exported functions, keep that.
- If a comment contradicts the code, the comment is a bug. Say so instead of silently deleting it.
