# React Patterns & Rules Cheat-Sheet

## Re-renders

- State update is the **only** source of re-renders
- When a component re-renders, **all children re-render** (down the tree, never up)
- Props changing does NOT trigger re-renders — only `React.memo` checks props
- Hooks hide state: any state update inside a hook re-renders the host component, even if the value isn't returned

### Moving State Down

Extract state + its consumers into a smaller component so unrelated siblings don't re-render.

### Children as Props

Elements passed as `children` don't re-render when the wrapper's state changes — React sees the same object reference.

```jsx
<ScrollableArea><SlowComponent /></ScrollableArea>
// SlowComponent won't re-render when ScrollableArea's state changes
```

---

## Elements vs Components

- **Component**: function returning elements — `const A = () => <B />`
- **Element**: descriptor object — `<B />` → `{ type: B, props: {...} }`
- Elements are cheap objects; rendering only happens when returned from a component

### Elements as Props

Prefer passing elements over config props:

```jsx
<Button icon={<Loading color="white" size="large" />} />
```

---

## Render Props

Use when a parent needs to pass its own state into children:

```jsx
<Button renderIcon={(props) => <Icon {...props} />} />
```

Hooks replaced most use cases; render props remain useful when logic is tied to a DOM element.

---

## Memoization

### When It Matters

1. Value is in another hook's dependency array
2. Component is wrapped in `React.memo`
3. Value is passed to a memoized child

### Rules

- `useCallback`/`useMemo` on props is **useless** without `React.memo` on the child
- `React.memo` breaks when: spreading unknown props, passing non-memoized children, or using non-memoized values from custom hooks
- Memoize children too: `const content = useMemo(() => <div>text</div>, [])`
- `useMemo` for calculations: measure first — most JS is faster than a re-render

---

## Reconciliation & Keys

### How React Compares

- Same `type` reference → re-render (update)
- Different `type` reference → unmount old, mount new

### Never Define Components Inside Components

The `type` reference changes every render → unmount/remount on each render.

### Key Rules

- Identifies elements in arrays between re-renders
- Use stable IDs, **not array indices** (indices break memoization on reorder)
- Different `key` on same component forces remount (useful for state reset)

```jsx
<Input key={isCompany ? "company" : "person"} />
```

---

## Context Performance

- All consumers re-render when context value changes
- **Split contexts** by update frequency (data vs API/dispatch)
- Memoize context values: `useMemo(() => ({ data, handlers }), [data, handlers])`

---

## Refs

- Don't trigger re-renders on mutation
- Updates are synchronous and mutable (vs state: async, immutable)
- Use for: DOM access, mutable values across renders, escaping stale closures

### Closure Escape Pattern

Store latest callback in a ref, create debounced/throttled wrapper once with `useMemo(() => ..., [])`.

---

## useLayoutEffect vs useEffect

- `useLayoutEffect`: runs synchronously **before** paint — use for DOM measurement to avoid flicker
- `useEffect`: runs **after** paint
- `useLayoutEffect` doesn't run on the server — gate with `useEffect(() => setShouldRender(true), [])`

---

## Portals

- Escape CSS stacking context issues (z-index, transforms)
- `createPortal(children, domNode)`
- React lifecycle follows component tree; DOM/CSS follows actual DOM position
- Native form submit bubbles through DOM tree, not React tree

---

## Data Fetching

- Use `Promise.all` for parallel requests; avoid sequential `await` waterfalls
- Chrome: max 6 parallel requests per host — fire critical ones first
- Nest data providers at app root to parallelize independent fetches

### Race Conditions

Three patterns to prevent stale data:

1. **Cleanup boolean**: `let isActive = true; return () => { isActive = false; }`
2. **Ref guard**: compare `ref.current` with response ID before setting state
3. **AbortController**: `return () => controller.abort()`

---

## Error Handling

- `ErrorBoundary` catches: render errors, lifecycle errors
- Does **NOT** catch: event handlers, async code, errors in the boundary itself
- Catch async errors by re-throwing inside a state updater:

```jsx
const [, setState] = useState();
const throwError = (e) => setState(() => { throw e; });
```

---

## Debouncing

- Store latest callback in a ref
- Create the debounced function once via `useMemo(() => debounce(...), [delay])`
- Ref ensures the debounced function always calls the latest closure

---

## State Management

### Categorize First, Choose Tools Second

| Category | Tool |
|----------|------|
| Remote data | TanStack Query / SWR |
| URL state | TanStack Router search params / nuqs |
| Shared state | Context (1-2 states) → Zustand (if more) |
| Local state | useState / useReducer |

Most "global state" is remote data or URL state in disguise.

### Shared State Escalation

1. Prop drilling (2-3 levels max)
2. Context API (few shared states, acceptable re-render scope)
3. Zustand (selective subscriptions, no provider, minimal API)

### URL as State

Store in URL when state should survive refresh, be shareable, or represents navigation/filter context.

### Avoid

- Monolithic state libraries for simple needs
- Signals/observables (mental model mismatch with React)
- Heavy boilerplate

---

## Quick Reference

### Performance Checklist

1. Move state close to where it's used
2. Pass slow components as children/props to stateful wrappers
3. `React.memo` only with fully memoized props
4. Measure before optimizing

### Common Pitfalls

- Defining components inside components
- Array index as key with dynamic lists
- `useCallback`/`useMemo` without `React.memo`
- Spreading unknown props to memoized components
- Stale closures in callbacks/effects
