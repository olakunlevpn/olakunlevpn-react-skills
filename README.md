# React & Next.js Coding Standards

[![GitHub stars](https://img.shields.io/github/stars/olakunlevpn/olakunlevpn-react-skills?style=flat-square)](https://github.com/olakunlevpn/olakunlevpn-react-skills/stargazers)
[![License: MIT](https://img.shields.io/badge/License-MIT-brightgreen.svg?style=flat-square)](LICENSE.md)

Professional coding standards for building production-grade React and Next.js applications. Opinionated rules extracted from the best open-source React projects and years of building at scale.

This is an **agent skill** that works with Claude Code, Cursor, Cline, Gemini CLI and 40+ other AI coding agents. Once installed, your agent will follow these standards automatically when writing React code.

## Installation

```bash
npx skills add olakunlevpn/olakunlevpn-react-skills
```

That's it. The skill is now active for all your React projects.

## What It Does

Once installed, your AI coding agent automatically follows battle-tested React conventions. Here's what that looks like:

**Components use variants with `cva`, not conditional class strings:**

```tsx
const buttonVariants = cva(
  "inline-flex items-center justify-center rounded-md font-medium",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground",
        destructive: "bg-destructive text-white",
        outline: "border border-input bg-background",
      },
      size: {
        default: "h-9 px-4",
        sm: "h-8 px-3 text-xs",
        lg: "h-10 px-6",
      },
    },
    defaultVariants: { variant: "default", size: "default" },
  }
)

function Button({ className, variant, size, ...props }:
  React.ComponentProps<"button"> & VariantProps<typeof buttonVariants>
) {
  return <button className={cn(buttonVariants({ variant, size, className }))} {...props} />
}
```

**Server Components by default, `"use client"` only on interactive leaves:**

```tsx
// app/dashboard/page.tsx -- Server Component, no directive needed
export default async function DashboardPage() {
  const [stats, orders] = await Promise.all([getStats(), getOrders()])

  return (
    <div>
      <StatsCards stats={stats} />
      <Suspense fallback={<OrdersSkeleton />}>
        <RecentOrders orders={orders} />
      </Suspense>
    </div>
  )
}
```

**Custom hooks return stable references and use options objects:**

```tsx
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value)

  useEffect(() => {
    const handler = setTimeout(() => setDebouncedValue(value), delay)
    return () => clearTimeout(handler)
  }, [value, delay])

  return debouncedValue
}
```

**Tests use Vitest + RTL with behavioral assertions, not snapshots:**

```tsx
it("should display validation errors for empty fields", async () => {
  render(<CreatePostForm />)

  await userEvent.click(screen.getByRole("button", { name: "Submit" }))

  await waitFor(() => {
    expect(screen.getByText("Title is required")).toBeInTheDocument()
  })
})
```

## How to Use

The skill auto-triggers when your agent detects React or Next.js work. You can also invoke it directly:

**Write a new component:**
```
Create a Dialog component with compound pattern, variants, and full accessibility
```

**Create a custom hook:**
```
Create a useLocalStorage hook that syncs across tabs
```

**Set up state management:**
```
Set up Zustand store for the shopping cart with selectors
```

**Write tests:**
```
Write Vitest tests for the CreatePostForm component
```

**Review existing code:**
```
Review this component for performance issues and suggest fixes
```

**Set up data fetching:**
```
Create a Server Action for creating posts with Zod validation and useActionState
```

**Project structure:**
```
Set up the folder structure for a Next.js App Router project with auth, dashboard, and settings sections
```

The agent will automatically follow all the standards in this skill -- Server Components by default, proper TypeScript types, cva variants, cn() merging, Pest testing patterns, and everything else.

## What's Covered

The skill covers 10 areas of React development:

**Components** -- Server vs client boundary, function components, composition over configuration, compound components, variants with cva, Tailwind-first styling, data-slot attributes, className merging

**TypeScript** -- Extending native element props, intersection types for variants, generics with defaults, function overloads, discriminated unions, Zod schema inference

**Custom Hooks** -- Options object pattern, stable references, useSyncExternalStore for external state, controlled/uncontrolled dual-mode, naming conventions

**State Management** -- Decision tree (local to global), Context splitting, Zustand selectors, Jotai atoms, TanStack Query for server state

**Data Fetching** -- Server Component async/await, parallel fetching, Server Actions, useActionState, useOptimistic, useFormStatus

**Performance** -- Architecture-first optimization, memoization only after profiling, code splitting, list virtualization, React Compiler awareness, asset optimization

**Error Handling** -- Error boundaries per route segment, Suspense for loading, class-based and library ErrorBoundary patterns

**Project Structure** -- Next.js App Router conventions, feature colocation, component/hook/action organization, barrel file rules

**Testing** -- Vitest + React Testing Library, hooks via real components, behavioral assertions, fake timers, render count tracking, StrictMode wrapping

**Accessibility** -- ARIA attributes, keyboard handling, focus trapping, useId for unique IDs, dev-only a11y warnings

## Quick Reference

The patterns you'll use most:

**State management decision tree:**
1. Local UI state -- `useState`
2. Derived state -- compute during render
3. Shared across siblings -- lift to parent
4. Shared across tree -- Context (low-frequency)
5. High-frequency global -- Zustand with selectors
6. Server state -- TanStack Query

**Performance rules of thumb:**
- Architecture fixes before memoization
- `React.memo` only after profiling confirms benefit
- `useMemo` only for calculations over 1ms
- `useCallback` only for props to memo-wrapped children
- Server Components ship zero JS to the browser

## Checklist

Before submitting any React code:
- Server Component by default, `"use client"` only on interactive leaves
- No `forwardRef` -- ref as a prop (React 19)
- Props spread last, className merged via `cn()`
- TypeScript types extend native element props
- Custom hooks return stable references
- State as local as possible, context split by domain
- Data fetched in Server Components when possible
- Memoization only after profiling
- Error boundaries on route segments and features
- Tests use Vitest + RTL, behavioral assertions
- Accessibility: keyboard, ARIA, focus management
- Stable keys on lists, virtualized if over 100 items

## Files

```
SKILL.md        # Main skill file (loaded by AI agents)
REFERENCE.md    # Detailed code examples for every pattern
```

## Contributing

Found a pattern that should be included? Open an issue or submit a pull request. Keep examples generic and framework-focused.

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
