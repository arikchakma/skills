---
name: react-component
description: write react components the way this codebase writes them
argument-hint: "What the component is for"
---

Conventions for writing or changing a React component.

## Look before you write

Search the codebase first. The component can be here already: a button, a modal, an empty state, a spinner. Reuse it. If it is close but not exact, add a prop to it.

If nothing fits, write a new component.

## Props

- Give every component one `props` argument.
- Type it with a named `XxxProps` type.
- Destructure `props` inside the function body.
- Apply this shape to small private components too.

```tsx
type ComponentProps = {
  /* ... */
};

export function Component(props: ComponentProps) {
  const {
    /* ... */
  } = props;
}
```

## One component per file

Name the file after the component. A file that holds five components must become five files.

One exception exists. A private component can stay in the file of the component that uses it, when it meets all three conditions:

- It serves only the component in that file.
- It is not exported.
- It is under 20 lines.

If it grows past 20 lines, or a second file needs it, move it to its own file.

## Tailwind

Never put Tailwind classes in a constant.

```tsx
// Bad
const buttonClasses = 'rounded-md px-3 py-2 text-sm font-medium';

// Good - inline
<button className="rounded-md px-3 py-2 text-sm font-medium" />
```

A long inline string is correct. The classes stay next to the element that they style, so you read the element and its styles in one place. A constant hides the styles, and it is less reusable than it looks.

If the same classes appear on the same kind of element three or more times, write a component. Do not write a constant.

Merge conditional classes with `cn` (`https://github.com/shadcn-ui/cn`). Do not build the string with a template literal.

### Stay on the scale

Tailwind has a spacing scale in steps of `0.25rem`, which is 4px. It includes the half steps: `0.5` is 2px, `1.5` is 6px, `2.5` is 10px. The scale covers almost every gap, padding, and margin that a design needs.

```tsx
// Bad
<div className="mt-[13px] gap-[7px] text-[15px] rounded-[5px]" />

// Good
<div className="mt-3 gap-2 text-sm rounded-md" />
```

Pick the nearest step. A 1px or 2px difference from the mock is not worth an arbitrary value. But if it's super necessary for the design system you can use p-1.25 (5px) or their define variable in the stylesheet (`@theme`).

The same rule covers color, font size, radius, and width. Use the token, not the raw value.

An arbitrary value is permitted when a real constraint sets the number, and the scale has no step for it. Examples: the height of a third-party widget, the exact size of an asset, a magic number from a vendor stylesheet. Write it inline, and put a comment above the element that names the constraint.

## Variants

Use [`class-variance-authority`](https://www.npmjs.com/package/class-variance-authority) for a component that has variants. Do not write your own map of class strings. Do not chain ternary operators in `className`.

```tsx
import { cva, type VariantProps } from 'class-variance-authority';

const button = cva('inline-flex items-center rounded-md font-medium', {
  variants: {
    variant: {
      primary: 'bg-black text-white hover:bg-black/90',
      ghost: 'bg-transparent text-black hover:bg-black/5',
    },
    size: {
      sm: 'h-8 px-3 text-sm',
      md: 'h-10 px-4 text-sm',
    },
  },
  defaultVariants: { variant: 'primary', size: 'md' },
});

type ButtonProps = VariantProps<typeof button> & {
  children: React.ReactNode;
};

export function Button(props: ButtonProps) {
  const { variant, size, children } = props;

  return <button className={button({ variant, size })}>{children}</button>;
}
```

## Match the existing design

A new component must look like it was always here. Open two or three screens near it before you style it.

Copy what those screens already use: spacing scale, colors, type sizes, font weights, border radius, borders, shadows, and empty states. Take the tokens and theme variables from the project. Do not write a raw hex color.

If the user asked for a different design, build what the user asked for.

## Do not reinvent a design system

Use [Base UI](https://base-ui.com/) (`@base-ui-components/react`) for every component that has behavior: tabs, dialog, select, popover, tooltip, menu, accordion, slider. Style it with Tailwind.

These components carry keyboard support, focus traps, and ARIA attributes. That work takes days, and one missed case breaks it. Do not write your own.

If Base UI has no component for it, build it by hand.

## Rules

- Do not put comments inside JSX or inside JSX prop expressions. Put the note above the element. The `comments` skill has the details.
- Match the file that you are in. If the folder uses `interface`, use `interface`, mostly try to match the existing style.
- Do not add a prop for a case that nobody asked for.
