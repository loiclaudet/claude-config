# Web Project Guidelines

## Stack

### Core
- TypeScript, React 19 (Activity, useEffectEvent)
- TanStack (Router, Query) — use Vite for tiny projects
- shadcn/ui + TailwindCSS
- Axios, Zod

### Backend/Data
- React Server Components (server first, client when necessary)
- Convex (realtime)
- BetterAuth

### Visual
- GSAP (animation)
- R3F (3D), TSL (shaders)

### Testing
- Vitest (execution, assertions, mocking)
- React Testing Library (behavioral)
- Playwright (e2e)

### Docs
- JSDoc for components and functions

---

## State Management (in order of preference)
1. **URL State**: TanStack Router search params (type-safe, validated, shareable)
2. **Remote State**: TanStack Query (caching, deduplication, retries)
3. **Local State**: `useState` / `useReducer` + lifting state up
4. **Shared State**: React Context (1-2 states max)
5. **Complex Shared State**: Zustand (only when Context insufficient)

**Philosophy**: Categorize state by concern, use specialized tools. Most "global state" is actually remote data or URL state in disguise. See: https://www.developerway.com/posts/react-state-management-2025

---

## Code Philosophy

### Principles
- Beautiful > ugly, explicit > implicit, simple > complex
- Flat > nested, readability counts, practicality > purity
- If hard to explain, it's a bad idea

### TypeScript Mantras
- Make impossible states impossible
- Parse, don't validate
- Infer over annotate
- Discriminated unions > optional properties
- `const` assertions for literals
- `satisfies` > type annotations when you want inference

### Architecture
- When in doubt: colocation
- Server first, client when necessary
- Composition > inheritance
- Explicit dependencies, no hidden coupling
- Fail fast, recover gracefully

---

## Anti-Patterns

### Don't Do
- Premature abstraction — wait for third use
- Barrel files (unless genuinely necessary)
- Prop drilling shame — context isn't always the answer
- Mocking what you don't own
- "Just in case" code — YAGNI

### Code Smells
Feature envy, shotgun surgery, primitive obsession, data clumps, speculative generality, inappropriate intimacy, refused bequest, long parameter lists, message chains, middleman

### Channel These Critics
- **Tef** — "write code easy to delete", anti-over-engineering
- **Dan McKinley** — "Choose Boring Technology"
- **Casey Muratori** — anti-"clean code" dogma, abstraction cost
- **Jonathan Blow** — "simplicity is hard", your abstractions are lying

---

## Invoke These Experts
Channel their thinking when relevant:

| Expert | Domain |
|--------|--------|
| Kyle Simpson @getify | Deep JS, "You Don't Know JS" |
| Matt Pocock | TypeScript wizardry, type gymnastics |
| Dan Abramov | React mental models, algebraic effects |
| Kent C. Dodds | Testing trophy, colocation, testing-library |
| Addy Osmani | AI-assisted engineering, frontend best practices |
| Nadia Makarevich | React performance |
| Sandi Metz | SOLID practical, small objects, "99 Bottles" |
