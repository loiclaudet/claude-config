---
description: Detect project tooling and create .claude/settings.local.json (linting hooks) + .claude/CLAUDE.md (code conventions)
user-invocable: true
---

Inspect this project and set up Claude Code integration.

## Part 1 — PostToolUse Hooks (linting/formatting)

1. Read `package.json` to detect:
   - **prettier** in dependencies/devDependencies
   - **eslint** in dependencies/devDependencies
   - **biome** (`@biomejs/biome`) in dependencies/devDependencies
2. If `.claude/settings.local.json` already exists, read it and merge — don't overwrite existing settings
3. Create/update `.claude/settings.local.json` with a PostToolUse hook matching what the project uses:
   - **Prettier + ESLint**: run both on Edit|Write for js/jsx/ts/tsx/css/html files:
     ```json
     {
       "hooks": {
         "PostToolUse": [
           {
             "matcher": "Edit|Write",
             "hooks": [
               {
                 "type": "command",
                 "command": "filepath=$(cat - | jq -r '.tool_input.file_path // empty'); case \"$filepath\" in *.js|*.jsx|*.ts|*.tsx|*.css|*.html) cd \"$(dirname \"$filepath\")\" && npx prettier --write \"$filepath\" 2>/dev/null; npx eslint --fix \"$filepath\" 2>/dev/null; exit 0 ;; *) exit 0 ;; esac"
               }
             ]
           }
         ]
       }
     }
     ```
   - **Prettier only** (no ESLint): same but without the eslint --fix call
   - **ESLint only** (no Prettier): same but without the prettier --write call
   - **Biome**: run `biome check --write` instead:
     ```json
     {
       "hooks": {
         "PostToolUse": [
           {
             "matcher": "Edit|Write",
             "hooks": [
               {
                 "type": "command",
                 "command": "filepath=$(cat - | jq -r '.tool_input.file_path // empty'); case \"$filepath\" in *.js|*.jsx|*.ts|*.tsx|*.css|*.html|*.json) cd \"$(dirname \"$filepath\")\" && npx biome check --write \"$filepath\" 2>/dev/null; exit 0 ;; *) exit 0 ;; esac"
               }
             ]
           }
         ]
       }
     }
     ```
   - **Neither found**: inform the user that no formatter/linter was detected, don't create the file
4. Show what was created and confirm the hook is active

## Part 2 — Project Conventions (.claude/CLAUDE.md)

1. Detect the project's stack by reading `package.json`:
   - **React project**: has `react` in dependencies
   - **Web project**: has any of `@tanstack/react-router`, `@tanstack/react-query`, `tailwindcss`, `next`, `vite` in dependencies/devDependencies
2. If React or web stack detected, create `.claude/CLAUDE.md` (if it doesn't already exist) with the relevant conventions:

```markdown
# Code Conventions
- Name files only in kebab-case
- Never export default (unless framework requires it)
- Never use barrel files
- Never use `any` — use `unknown`
- Never use enums — use union types
- Prefer `type` over `interface`
- Use arrow functions for component exports: `export const Foo = () =>`
- Use `function` keyword only for hoisted helpers inside components
- Always use TailwindCSS `className` — never `StyleSheet.create` or inline styles

# Reference Skills
- `/react` — React patterns (re-renders, memoization, reconciliation, state management)
- `/web` — Web stack guidelines (state management, architecture, anti-patterns)
```

3. If `.claude/CLAUDE.md` already exists, don't overwrite — inform the user it already exists
4. If no React/web stack detected, skip this step and inform the user

## Summary

After completing both parts, show the user:
- What hooks were configured (or that none were needed)
- Whether project conventions were created (or that the project isn't a React/web project)
