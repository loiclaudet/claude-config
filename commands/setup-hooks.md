Inspect this project's linting/formatting setup and create `.claude/settings.local.json` with appropriate PostToolUse hooks.

Steps:
1. Read `package.json` (and any lockfiles if needed) to detect:
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
