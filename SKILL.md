---
name: olakunlevpn-react-skills
description: Use when writing, reviewing, or refactoring React or Next.js code — components, hooks, state management, testing, performance, TypeScript patterns, server components, or project structure. Always apply these standards to all React work.
---

# React & Next.js Coding Standards

Comprehensive coding standards for building production-grade React and Next.js applications. Follow them exactly. For detailed code examples, see REFERENCE.md.

## When to Apply

Apply these standards to ALL React and Next.js work:
- Writing new components (server and client)
- Creating custom hooks
- State management setup and usage
- Writing tests (Vitest + React Testing Library)
- TypeScript types and interfaces for components
- Performance optimization
- Project structure decisions
- Form handling, data fetching, error boundaries

---

## 1. Component Rules

### File Structure
- One component per file. File name matches component name in PascalCase.
- `"use client"` only on the smallest interactive leaf components, never on large parent trees.
- Default to Server Components. Only add `"use client"` when you need state, effects, event handlers, or browser APIs.

### Function Components
- Plain function declarations or arrow functions. No class components.
- No `React.forwardRef` -- pass `ref` directly as a prop (React 19+).
- Props typed inline with `React.ComponentProps<"element">` for extending native elements.
- Props spread last: `{...props}` always after default props so consumers can override.

### Composition Over Configuration
- Accept `children` as the primary composition mechanism.
- Use compound components for complex UI (Modal, Dialog, Accordion).
- Share implicit state via Context between compound sub-components.
- Use `asChild` / Slot pattern for polymorphic rendering without extra DOM nodes.

### Styling
- Tailwind-first with `cn()` utility (`clsx` + `tailwind-merge`).
- Use `cva` (class-variance-authority) for variant systems.
- Design tokens via CSS custom properties: `bg-primary`, `text-muted-foreground`.
- Dark mode via Tailwind `dark:` variant. No separate dark theme files.
- `data-slot="component-name"` attribute on every component root for CSS targeting.
- `className` always merged, never replaced: `cn(defaultClasses, className)`.
- `aria-invalid:` variant for form validation styling.

### Accessibility
- All interactive components need keyboard handling and ARIA attributes.
- `aria-labelledby`, `aria-describedby`, `aria-expanded`, `aria-haspopup` on dialogs/menus.
- IDs generated via `useId()` for uniqueness.
- Focus trapping in modals. Return focus to trigger on close.
- Dev-only warnings for missing required a11y elements (title, description).
- `type="button"` on non-submit buttons to prevent form submission.

---

## 2. TypeScript Patterns

- Extend native element props: `React.ComponentProps<"button"> & CustomProps`
- Intersection types for variants: `React.ComponentProps<"button"> & VariantProps<typeof variants>`
- Generics with defaults for reusable hooks: `<T = unknown, E = Error>`
- Function overloads for different return types based on input config.
- `NoInfer<T>` to prevent inference from return position.
- Use `as const` for literal types. Use discriminated unions for tagged types.
- Export types alongside components. Colocate types with their component.
- Zod for runtime validation schemas. Infer types with `z.infer<typeof schema>`.

---

## 3. Custom Hook Rules

### Design
- Single options object parameter, not positional args. More readable, extensible, TypeScript-friendly.
- Return stable references: wrap returned functions in `useCallback`, objects in `useMemo`.
- Use `useSyncExternalStore` for subscribing to external state (localStorage, matchMedia, etc.).
- Handle controlled/uncontrolled dual-mode with a `useControllableState` pattern.

### Naming
- `use` prefix always: `useDebounce`, `useMediaQuery`, `useLocalStorage`.
- Name describes what it manages, not how it works.
- Return a tuple `[value, setter]` for simple state, or a named object for complex state.

### Common Patterns
```
useDebounce(value, delay)        -- timer cleanup in useEffect
useLocalStorage(key, initial)    -- useSyncExternalStore + StorageEvent
useMediaQuery(query)             -- useSyncExternalStore + matchMedia
useControllableState(prop, default, onChange) -- controlled/uncontrolled
```

---

## 4. State Management

### Decision Tree
1. **Local UI state** -- `useState`. Keep state as local as possible.
2. **Derived state** -- Compute during render. No state needed.
3. **Shared across siblings** -- Lift to nearest common parent.
4. **Shared across tree** -- Context for low-frequency updates (theme, auth, locale).
5. **High-frequency global** -- Zustand with selectors, or Jotai atoms.
6. **Server state** -- TanStack Query (caching, revalidation, optimistic updates).

### Context Rules
- Split context by domain. One context per concern (theme, auth, cart).
- Place providers as deep as possible in the tree.
- Context providers must be Client Components in Next.js.
- `<ThemeContext value="dark">` (React 19, no `.Provider` needed).

### Zustand / Jotai
- Zustand: selector pattern prevents unnecessary re-renders. `useStore(store, selector)`.
- Jotai: atomic model, each atom is independent. `useAtom(countAtom)`.
- Both bridge external state into React via subscription + forced re-render.

---

## 5. Data Fetching

### Server Components (preferred)
- Fetch data directly in Server Components with `async/await`.
- Use `cache()` for request deduplication.
- Parallel fetching: initiate all requests before awaiting any.

### Server Actions
- `"use server"` functions for mutations.
- Use `useActionState` for form submissions: returns `[data, action, isPending]`.
- `useOptimistic` for instant UI feedback during async operations.
- `useFormStatus` for pending state in submit buttons.

### Client-Side
- TanStack Query for client-side server state.
- Single options object: `useQuery({ queryKey, queryFn })`.
- Never fetch in `useEffect` for data that should be server-fetched.

---

## 6. Performance

### Architecture First (most impactful)
- Accept JSX as `children` -- wrapper re-renders don't cascade.
- Keep state local. Don't lift higher than necessary.
- Split context by domain. One update doesn't affect unrelated consumers.
- Server Components ship zero JS to the browser.

### Memoization (only when measured)
- `React.memo` only after profiling confirms a benefit. Not a default.
- `useMemo` for calculations over 1ms (measure with `console.time`).
- `useCallback` only when passing functions to `memo`-wrapped children.
- Updater functions eliminate dependencies: `setTodos(prev => [...prev, item])`.
- React Compiler (v1.0) handles most memoization automatically in new projects.

### Code Splitting
- `React.lazy()` + `Suspense` for route-based splitting.
- Dynamic `import()` for heavy components loaded on interaction.
- `next/dynamic` with `ssr: false` for browser-only libraries.

### Lists and Rendering
- Virtualize lists over 100 items (`react-window` or `react-virtuoso`).
- Stable `key` props. Never use array index as key for dynamic lists.
- `useTransition` for expensive updates that should not block input.
- `useDeferredValue` to show stale content while fresh content prepares.

### Assets
- `next/image` for automatic format negotiation, responsive sizing, lazy loading.
- `next/font` for automatic subsetting, no layout shift.
- Prefer `transform` and `opacity` for animations (GPU compositor thread).

---

## 7. Error Handling

### Error Boundaries
- Wrap route segments and independent features in Error Boundaries.
- Suspense handles loading. Error Boundary handles failure.
- Next.js: use `error.tsx` files per route segment.
- Provide a reset mechanism (`retry` button) in fallback UI.

### Patterns
- Class-based `ErrorBoundary` with `getDerivedStateFromError`.
- Or `react-error-boundary` package for simpler API.
- Dev-only runtime validation (guard with `process.env.NODE_ENV !== 'production'`).

---

## 8. Project Structure

### Next.js App Router
```
app/
  layout.tsx              # Root layout (Server Component)
  page.tsx                # Home page
  (auth)/
    login/page.tsx        # Grouped routes
    register/page.tsx
  dashboard/
    layout.tsx            # Nested layout
    page.tsx
    loading.tsx           # Suspense fallback
    error.tsx             # Error boundary
components/
  ui/                     # Reusable primitives (button, dialog, input)
  features/               # Feature-specific components
hooks/                    # Custom hooks
lib/                      # Utilities, API clients, config
types/                    # Shared TypeScript types
actions/                  # Server actions
```

### Rules
- Colocate files with their feature. Shared code in top-level directories.
- `components/ui/` for design system primitives. `components/features/` for domain logic.
- One hook per file in `hooks/`. Name matches the hook.
- Barrel files (`index.ts`) only for `components/ui/`. Avoid elsewhere.
- Server actions in `actions/` directory or colocated with their route.
- Types colocated with components unless shared across features.

---

## 9. Testing (Vitest + React Testing Library)

### Framework
- Vitest exclusively. Not Jest.
- React Testing Library for component tests.
- Test hooks by rendering real components that use them. Not `renderHook`.

### Naming
- Descriptive behavioral names: `it('should display error when form is invalid')`.
- Describe behavior, not implementation.

### Patterns
```typescript
// Render with providers
function renderWithProviders(ui: React.ReactElement) {
  return render(
    <QueryClientProvider client={queryClient}>
      <ThemeProvider>{ui}</ThemeProvider>
    </QueryClientProvider>
  )
}

// Behavioral assertions
expect(screen.getByText('Submit')).toBeInTheDocument()
expect(screen.getByRole('button')).toBeDisabled()

// Async
await waitFor(() => expect(screen.getByText('Loaded')).toBeInTheDocument())

// User events
await userEvent.click(screen.getByRole('button', { name: 'Save' }))

// State transitions (collector pattern)
const states: Result[] = []
function Wrapper() {
  const state = useQuery(options)
  states.push(state)
  return <div>{state.data}</div>
}
```

### Rules
- Behavioral assertions over snapshots. Test what the user sees.
- `vi.useFakeTimers()` for deterministic async tests.
- `act()` wrapping for synchronous state updates.
- Track render counts to verify optimization.
- Wrap in `<StrictMode>` to catch double-render bugs.
- Mock with `vi.fn()` and `vi.spyOn()`. Use `mockRestore()` in cleanup.

---

## 10. Event Handling

- Compose event handlers, never silently override: `composeEventHandlers(props.onClick, internalHandler)`.
- Consumer handler runs first, can `preventDefault()` to cancel internal handler.
- Use updater form of setState in callbacks to avoid stale closures.
- `onPointerDown` over `onClick` for drag and dismiss interactions.

---

## Summary Checklist

Before submitting any React code:
- [ ] Server Component by default, `"use client"` only on interactive leaves
- [ ] No `forwardRef` -- ref as a prop (React 19)
- [ ] Props spread last, className merged via `cn()`
- [ ] TypeScript types extend native element props
- [ ] Custom hooks return stable references
- [ ] State as local as possible, context split by domain
- [ ] Data fetched in Server Components when possible
- [ ] `React.memo` / `useMemo` / `useCallback` only after profiling
- [ ] Error boundaries on route segments and features
- [ ] Tests use Vitest + RTL, behavioral assertions, real component rendering
- [ ] Accessibility: keyboard handling, ARIA attributes, focus management
- [ ] Stable keys on lists, virtualized if over 100 items
