---
title: "Tailwind CSS in Practice: Tips I Use Every Day"
description: "A set of patterns, utilities, and best practices from Tailwind CSS that truly make a difference in day-to-day front-end development."
date: "2026-04-02"
tags: ["tailwind css", "css", "front-end"]
---

Tailwind CSS changed the way I write styles. After years alternating between plain CSS, CSS Modules, and styled-components, Tailwind has become my default choice for most projects. In this post I'll share the patterns I use most in my daily work.

## Why Tailwind?

The premise is simple: inline styles with superpowers. You write utility classes directly in your HTML/JSX and Tailwind generates only the CSS you actually use — no dead code.

The result is a much smaller CSS bundle and much greater visual consistency, since everything comes from a unified design system.

## Responsive Variants

Tailwind uses a mobile-first approach. The `sm`, `md`, `lg`, `xl` prefixes apply styles from that breakpoint upward:

```tsx
<div className="flex flex-col md:flex-row gap-4 md:gap-8">
  <aside className="w-full md:w-64">Sidebar</aside>
  <main className="flex-1">Content</main>
</div>
```

No manual media queries — everything inline and readable.

## Dark Mode

With `darkMode: "class"` in the config, you use the `dark:` prefix for dark theme styles:

```tsx
<div className="bg-white text-gray-900 dark:bg-gray-950 dark:text-gray-100">
  <p className="text-gray-600 dark:text-gray-400">
    Secondary text
  </p>
</div>
```

## The `cn()` Pattern With clsx and tailwind-merge

A common problem in Tailwind is class conflicts when combining props with default classes. The solution is the `cn()` function:

```ts
// lib/utils.ts
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

Practical use in components:

```tsx
interface ButtonProps {
  className?: string;
  variant?: "primary" | "ghost";
}

export function Button({ className, variant = "primary" }: ButtonProps) {
  return (
    <button
      className={cn(
        "px-4 py-2 rounded-md font-medium transition-colors",
        variant === "primary" && "bg-gray-900 text-white hover:bg-gray-700",
        variant === "ghost" && "bg-transparent hover:bg-gray-100",
        className
      )}
    >
      Button
    </button>
  );
}
```

`twMerge` resolves conflicts — if you pass `className="bg-red-500"`, it correctly overrides `bg-gray-900`.

## Group and Peer

`group` and `peer` are two of Tailwind's most powerful and underused utilities.

**group**: applies styles to children when the parent is hovered:

```tsx
<div className="group flex items-center gap-2 cursor-pointer">
  <span>See more</span>
  <ArrowRight className="size-4 transition-transform group-hover:translate-x-1" />
</div>
```

**peer**: applies styles to siblings based on an element's state:

```tsx
<input
  type="checkbox"
  className="peer hidden"
  id="toggle"
/>
<label
  htmlFor="toggle"
  className="peer-checked:bg-green-500 bg-gray-200 cursor-pointer px-4 py-2 rounded"
>
  Toggle
</label>
```

## Custom Animations in Config

For animations beyond the defaults, add to `tailwind.config.ts`:

```ts
export default {
  theme: {
    extend: {
      keyframes: {
        fadeIn: {
          from: { opacity: "0", transform: "translateY(8px)" },
          to: { opacity: "1", transform: "translateY(0)" },
        },
      },
      animation: {
        fadeIn: "fadeIn 0.3s ease-out",
      },
    },
  },
};
```

Usage:

```tsx
<div className="animate-fadeIn">Content with fade</div>
```

## Arbitrary Values

When the design system doesn't have the exact value you need, use brackets:

```tsx
<div className="top-[72px] w-[calc(100%-2rem)] bg-[#1a1a2e]">
  Content
</div>
```

Use sparingly — if you're relying on arbitrary values a lot, it's probably worth customizing the config instead.

## Final Tips

- **Install the Tailwind extension for VS Code** — the autocomplete is a game changer
- **Use `@layer components` for repetitive patterns** — avoids duplicating dozens of classes
- **Don't overuse `@apply`** — it goes against the utility philosophy and makes code harder to read
- **Prefer `gap` over `margin` in flex/grid layouts** — more predictable and easier to maintain

Tailwind isn't for everyone, but for those who embrace the utility-first philosophy, it's one of the most productive tools in the front-end ecosystem.