---
name: function-params
description: decide when a function takes one named options object and when it keeps positional parameters
argument-hint: "The function or file to check"
---

Rules for the parameters of a function, a hook, or a callback prop.

## Use one options object when

- The function has three or more parameters.
- Two parameters have the same type, so a caller can swap them and the type checker does not see it.
- A parameter is a boolean or a flag.
- A parameter is optional, or has a default.

```ts
// Bad - which string is which? Which flag is which?
findDomain(workspace.id, domainId);
updateTracking(domain, true, false);

// Good
findDomain({ workspaceId: workspace.id, domainId });
updateTracking({ domain, openTracking: true, clickTracking: false });
```

## Keep positional parameters when

- The function has one parameter.
- The function has two parameters of different types, and the name makes the order obvious: `chunk(items, size)`, `download(blob, filename)`.
- The function copies a signature that most codebases share: `useDebounce(value, delay)`, `useInterval(callback, delay)`.

## Shape

- Give the object a named type. Name it after the function: `findDomain` takes `FindDomainOptions`.
- If the function name already ends in `Options` (a query options factory), end the type in `Params`: `listEmailsOptions` takes `ListEmailsParams`.
- Destructure inside the body. Put defaults in the destructuring.

```ts
type FindDomainOptions = {
  workspaceId: string;
  domainId: string;
};

export async function findDomain(options: FindDomainOptions) {
  const { workspaceId, domainId } = options;
  // ...
}
```

Use shorthand at the call site when the variable has the same name as the option: `findDomain({ workspaceId, domainId })`.

If the last parameter is already an options object, merge the other parameters into it. Do not nest it.

```ts
// Bad
formatDayRange({ range, options: { year: false } });

// Good
formatDayRange({ range, year: false });
```

## Callback props

The same rules apply to a callback that a component takes as a prop. If the callback would receive two values of the same type, pass one object.

```tsx
type EmailDeliveryCardProps = {
  onCopy: (options: CopyValueOptions) => void;
};

onCopy({ value: email.id, label: 'Email ID' });
```

## Do not change

- A callback whose signature a library sets. The library calls it with positional arguments: `(c, next)` middleware, `onSuccess(data, variables)`, `onError(error, variables, context)`, `Array.prototype.reduce` callbacks, editor `handleKeyDown(view, event)`, a logger interface method.
- Vendored code that you copied from another project.

## Refactor a codebase

Change many functions at once with a codemod that uses the TypeScript type checker. A text search finds the wrong functions when two functions have the same name in different files.

1. List every function, arrow function, and object method that has two or more parameters. Apply the rules above to each one.
2. Back up the working tree if the changes are not committed.
3. Rewrite the signatures. Rewrite each call site that the checker resolves to that declaration.
4. At a call site, add only insertions (`{`, `name:`, `}`). Do not replace the full argument list, because a nested call inside it gets its own edit.
5. Look for references that are not calls: a function passed as a prop or a callback. Change the type of the prop too.
6. Run the type checker and the tests.

## Rules

- Remove a parameter that no caller passes. Do not move it into the options object.
- Do not add an option for a case that nobody asked for.
