# React & Next.js Coding Standards -- Detailed Reference

Complete code examples for every pattern in SKILL.md.

## Component with Variants

```tsx
import { cva, type VariantProps } from "class-variance-authority"
import { cn } from "@/lib/utils"
import { Slot } from "@radix-ui/react-slot"

const buttonVariants = cva(
  "inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors focus-visible:ring-[3px] focus-visible:ring-ring/50 disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        destructive: "bg-destructive text-white hover:bg-destructive/90",
        outline: "border border-input bg-background hover:bg-accent",
        ghost: "hover:bg-accent hover:text-accent-foreground",
      },
      size: {
        default: "h-9 px-4 py-2",
        sm: "h-8 rounded-md px-3 text-xs",
        lg: "h-10 rounded-md px-6",
        icon: "h-9 w-9",
      },
    },
    defaultVariants: { variant: "default", size: "default" },
  }
)

function Button({
  className,
  variant,
  size,
  asChild = false,
  ...props
}: React.ComponentProps<"button"> &
  VariantProps<typeof buttonVariants> & {
    asChild?: boolean
  }) {
  const Comp = asChild ? Slot : "button"

  return (
    <Comp
      data-slot="button"
      className={cn(buttonVariants({ variant, size, className }))}
      {...props}
    />
  )
}

export { Button, buttonVariants }
```

## Compound Component Pattern

```tsx
import { createContext, useContext, useId } from "react"

type DialogContextValue = {
  open: boolean
  onOpenChange: (open: boolean) => void
  titleId: string
  descriptionId: string
}

const DialogContext = createContext<DialogContextValue | null>(null)

function useDialogContext() {
  const context = useContext(DialogContext)
  if (!context) throw new Error("Dialog components must be used within <Dialog>")
  return context
}

function Dialog({
  children,
  open,
  onOpenChange,
}: {
  children: React.ReactNode
  open: boolean
  onOpenChange: (open: boolean) => void
}) {
  const id = useId()

  return (
    <DialogContext
      value={{
        open,
        onOpenChange,
        titleId: `${id}-title`,
        descriptionId: `${id}-description`,
      }}
    >
      {children}
    </DialogContext>
  )
}

function DialogTrigger({ children, ...props }: React.ComponentProps<"button">) {
  const { onOpenChange } = useDialogContext()

  return (
    <button
      data-slot="dialog-trigger"
      type="button"
      onClick={() => onOpenChange(true)}
      {...props}
    >
      {children}
    </button>
  )
}

function DialogContent({ children, className, ...props }: React.ComponentProps<"div">) {
  const { open, onOpenChange, titleId, descriptionId } = useDialogContext()
  if (!open) return null

  return (
    <div
      data-slot="dialog-content"
      role="dialog"
      aria-labelledby={titleId}
      aria-describedby={descriptionId}
      className={cn("fixed inset-0 z-50", className)}
      {...props}
    >
      {children}
    </div>
  )
}

function DialogTitle({ children, ...props }: React.ComponentProps<"h2">) {
  const { titleId } = useDialogContext()
  return <h2 id={titleId} data-slot="dialog-title" {...props}>{children}</h2>
}

Dialog.Trigger = DialogTrigger
Dialog.Content = DialogContent
Dialog.Title = DialogTitle

export { Dialog }
```

## Custom Hook -- useDebounce

```tsx
import { useState, useEffect } from "react"

export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value)

  useEffect(() => {
    const handler = setTimeout(() => setDebouncedValue(value), delay)
    return () => clearTimeout(handler)
  }, [value, delay])

  return debouncedValue
}
```

## Custom Hook -- useLocalStorage

```tsx
import { useSyncExternalStore, useCallback } from "react"

export function useLocalStorage<T>(key: string, initialValue: T) {
  const subscribe = useCallback(
    (callback: () => void) => {
      window.addEventListener("storage", callback)
      return () => window.removeEventListener("storage", callback)
    },
    []
  )

  const getSnapshot = () => {
    const item = localStorage.getItem(key)
    return item ? (JSON.parse(item) as T) : initialValue
  }

  const getServerSnapshot = () => initialValue

  const value = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot)

  const setValue = useCallback(
    (newValue: T | ((prev: T) => T)) => {
      const resolved =
        typeof newValue === "function"
          ? (newValue as (prev: T) => T)(JSON.parse(localStorage.getItem(key) ?? "null") ?? initialValue)
          : newValue
      localStorage.setItem(key, JSON.stringify(resolved))
      window.dispatchEvent(new StorageEvent("storage", { key }))
    },
    [key, initialValue]
  )

  return [value, setValue] as const
}
```

## Custom Hook -- useControllableState

```tsx
import { useState, useCallback, useRef } from "react"

export function useControllableState<T>({
  prop,
  defaultProp,
  onChange,
}: {
  prop?: T
  defaultProp: T
  onChange?: (value: T) => void
}) {
  const [uncontrolledValue, setUncontrolledValue] = useState(defaultProp)
  const isControlled = prop !== undefined
  const value = isControlled ? prop : uncontrolledValue

  const setValue = useCallback(
    (nextValue: T | ((prev: T) => T)) => {
      if (isControlled) {
        const resolved =
          typeof nextValue === "function"
            ? (nextValue as (prev: T) => T)(prop as T)
            : nextValue
        onChange?.(resolved)
      } else {
        setUncontrolledValue(nextValue)
      }
    },
    [isControlled, prop, onChange]
  )

  return [value, setValue] as const
}
```

## Server Component with Data Fetching

```tsx
// app/dashboard/page.tsx (Server Component -- no "use client")
import { Suspense } from "react"

async function getStats() {
  const res = await fetch("https://api.example.com/stats", {
    next: { revalidate: 60 },
  })
  return res.json()
}

async function getRecentOrders() {
  const res = await fetch("https://api.example.com/orders?limit=10")
  return res.json()
}

export default async function DashboardPage() {
  // Parallel data fetching -- initiate before awaiting
  const statsPromise = getStats()
  const ordersPromise = getRecentOrders()

  const [stats, orders] = await Promise.all([statsPromise, ordersPromise])

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

## Server Action with useActionState

```tsx
// actions/create-post.ts
"use server"

import { z } from "zod"

const schema = z.object({
  title: z.string().min(1),
  content: z.string().min(10),
})

export async function createPost(prevState: any, formData: FormData) {
  const parsed = schema.safeParse({
    title: formData.get("title"),
    content: formData.get("content"),
  })

  if (!parsed.success) {
    return { errors: parsed.error.flatten().fieldErrors }
  }

  await db.post.create({ data: parsed.data })
  return { success: true }
}
```

```tsx
// components/create-post-form.tsx
"use client"

import { useActionState } from "react"
import { useFormStatus } from "react-dom"
import { createPost } from "@/actions/create-post"

function SubmitButton() {
  const { pending } = useFormStatus()
  return (
    <button type="submit" disabled={pending}>
      {pending ? "Creating..." : "Create Post"}
    </button>
  )
}

export function CreatePostForm() {
  const [state, action] = useActionState(createPost, null)

  return (
    <form action={action}>
      <input name="title" />
      {state?.errors?.title && <p>{state.errors.title[0]}</p>}
      <textarea name="content" />
      {state?.errors?.content && <p>{state.errors.content[0]}</p>}
      <SubmitButton />
    </form>
  )
}
```

## useOptimistic

```tsx
"use client"

import { useOptimistic } from "react"

export function TodoList({
  todos,
  addTodo,
}: {
  todos: Todo[]
  addTodo: (text: string) => Promise<void>
}) {
  const [optimisticTodos, addOptimistic] = useOptimistic(
    todos,
    (state, newTodo: string) => [...state, { id: crypto.randomUUID(), text: newTodo, pending: true }]
  )

  async function handleSubmit(formData: FormData) {
    const text = formData.get("text") as string
    addOptimistic(text)
    await addTodo(text)
  }

  return (
    <div>
      {optimisticTodos.map((todo) => (
        <div key={todo.id} className={todo.pending ? "opacity-50" : ""}>
          {todo.text}
        </div>
      ))}
      <form action={handleSubmit}>
        <input name="text" />
        <button type="submit">Add</button>
      </form>
    </div>
  )
}
```

## Error Boundary

```tsx
"use client"

import { Component, type ReactNode } from "react"

interface Props {
  children: ReactNode
  fallback: ReactNode | ((error: Error, reset: () => void) => ReactNode)
}

interface State {
  hasError: boolean
  error: Error | null
}

export class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false, error: null }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error }
  }

  reset = () => this.setState({ hasError: false, error: null })

  render() {
    if (this.state.hasError && this.state.error) {
      return typeof this.props.fallback === "function"
        ? this.props.fallback(this.state.error, this.reset)
        : this.props.fallback
    }
    return this.props.children
  }
}
```

## Zustand Store

```tsx
import { create } from "zustand"

interface CartState {
  items: CartItem[]
  addItem: (item: CartItem) => void
  removeItem: (id: string) => void
  total: () => number
}

export const useCartStore = create<CartState>((set, get) => ({
  items: [],
  addItem: (item) => set((state) => ({ items: [...state.items, item] })),
  removeItem: (id) =>
    set((state) => ({ items: state.items.filter((i) => i.id !== id) })),
  total: () => get().items.reduce((sum, item) => sum + item.price, 0),
}))

// Usage with selector -- only re-renders when items.length changes
function CartCount() {
  const count = useCartStore((state) => state.items.length)
  return <span>{count}</span>
}
```

## TanStack Query

```tsx
"use client"

import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query"

export function usePosts() {
  return useQuery({
    queryKey: ["posts"],
    queryFn: async () => {
      const res = await fetch("/api/posts")
      if (!res.ok) throw new Error("Failed to fetch posts")
      return res.json() as Promise<Post[]>
    },
    staleTime: 5 * 60 * 1000,
  })
}

export function useCreatePost() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: async (data: CreatePostData) => {
      const res = await fetch("/api/posts", {
        method: "POST",
        body: JSON.stringify(data),
      })
      return res.json()
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["posts"] })
    },
  })
}
```

## Event Handler Composition

```tsx
function composeEventHandlers<E>(
  externalHandler?: (event: E) => void,
  internalHandler?: (event: E) => void
) {
  return (event: E) => {
    externalHandler?.(event)
    if (!(event as any).defaultPrevented) {
      internalHandler?.(event)
    }
  }
}

// Usage in component
function DialogTrigger({ onClick, ...props }: React.ComponentProps<"button">) {
  const { onOpenChange } = useDialogContext()

  return (
    <button
      onClick={composeEventHandlers(onClick, () => onOpenChange(true))}
      {...props}
    />
  )
}
```

## cn() Utility

```tsx
import { clsx, type ClassValue } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

## Testing -- Component Test

```tsx
import { describe, it, expect, vi, beforeEach, afterEach } from "vitest"
import { render, screen, waitFor } from "@testing-library/react"
import userEvent from "@testing-library/user-event"

describe("CreatePostForm", () => {
  it("should display validation errors for empty fields", async () => {
    render(<CreatePostForm />)

    await userEvent.click(screen.getByRole("button", { name: "Create Post" }))

    await waitFor(() => {
      expect(screen.getByText("Title is required")).toBeInTheDocument()
    })
  })

  it("should submit successfully with valid data", async () => {
    render(<CreatePostForm />)

    await userEvent.type(screen.getByLabelText("Title"), "My Post")
    await userEvent.type(screen.getByLabelText("Content"), "This is the content of my post")
    await userEvent.click(screen.getByRole("button", { name: "Create Post" }))

    await waitFor(() => {
      expect(screen.getByText("Post created")).toBeInTheDocument()
    })
  })
})
```

## Testing -- Hook via Real Component

```tsx
import { describe, it, expect, vi } from "vitest"
import { render, screen, act } from "@testing-library/react"
import { StrictMode } from "react"
import { useCartStore } from "./use-cart-store"

function CartCounter() {
  const count = useCartStore((state) => state.items.length)
  const addItem = useCartStore((state) => state.addItem)

  return (
    <div>
      <span>count: {count}</span>
      <button onClick={() => addItem({ id: "1", name: "Test", price: 10 })}>
        add
      </button>
    </div>
  )
}

it("should increment count when item is added", async () => {
  render(
    <StrictMode>
      <CartCounter />
    </StrictMode>
  )

  expect(screen.getByText("count: 0")).toBeInTheDocument()

  await act(() => {
    screen.getByText("add").click()
  })

  expect(screen.getByText("count: 1")).toBeInTheDocument()
})
```

## Testing -- Async with Fake Timers

```tsx
import { describe, it, expect, vi, beforeEach, afterEach } from "vitest"
import { render, screen } from "@testing-library/react"

beforeEach(() => vi.useFakeTimers())
afterEach(() => vi.useRealTimers())

it("should show debounced value after delay", async () => {
  function TestComponent() {
    const [value, setValue] = useState("")
    const debounced = useDebounce(value, 500)

    return (
      <div>
        <input onChange={(e) => setValue(e.target.value)} />
        <span>debounced: {debounced}</span>
      </div>
    )
  }

  render(<TestComponent />)

  await userEvent.type(screen.getByRole("textbox"), "hello")

  expect(screen.getByText("debounced:")).toBeInTheDocument()

  await vi.advanceTimersByTimeAsync(500)

  expect(screen.getByText("debounced: hello")).toBeInTheDocument()
})
```

## Testing -- Performance (Render Count)

```tsx
it("should only re-render components that use changed state", () => {
  let counterRenders = 0
  let nameRenders = 0

  function Counter() {
    const count = useStore((s) => s.count)
    counterRenders++
    return <div>count: {count}</div>
  }

  function Name() {
    const name = useStore((s) => s.name)
    nameRenders++
    return <div>name: {name}</div>
  }

  render(
    <>
      <Counter />
      <Name />
    </>
  )

  act(() => useStore.setState({ count: 1 }))

  expect(counterRenders).toBe(2) // initial + update
  expect(nameRenders).toBe(1)   // initial only -- not affected
})
```
