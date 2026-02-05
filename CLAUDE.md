# User Preferences

## Dev Server
- Never start local dev servers automatically
- If a dev server is needed but not running, provide the command to run instead of launching it

## Code Style
- Prefer functional patterns over class-based when idiomatic for the language
- Use early returns to reduce nesting
- Prefer `jq` over `python3` for JSON parsing in shell commands
- Name files only in kebab-case
- Never export default, except when required by a framework (Expo Router, Next.js pages)
- Never use barrel files (index.ts re-exports)
- Never use `any` — use `unknown` instead
- Never use enums — use union types
- Prefer `type` over `interface`
- Use arrow functions for component exports: `export const Foo = () =>`
- Use `function` keyword only for hoisted helpers inside components
- Always use TailwindCSS `className` for styling — never use `StyleSheet.create` or inline style objects

## Communication
- Be direct and concise — skip preamble
- Don't explain code changes unless asked

## Skills
- `/react` — React patterns (re-renders, memoization, reconciliation, state management)
- `/web` — Web project guidelines (stack, architecture, anti-patterns)
