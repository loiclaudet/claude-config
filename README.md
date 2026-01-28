# Claude Code Config

My [Claude Code](https://docs.anthropic.com/en/docs/claude-code) configuration — settings, hooks, and skills.

## What's Inside

### `settings.json`
- **Notification hooks**: Sound alerts for permission prompts and idle state (10s delay, auto-cancelled if responded to). Push notifications via [ntfy.sh](https://ntfy.sh) for permission prompts only (idle uses local sound only to reduce noise)
- **Hook cancellation**: `PreToolUse` and `UserPromptSubmit` hooks that kill pending notifications when you respond in time

### `CLAUDE.md`
Global preferences applied to all projects.

### `skills/`
On-demand guidelines loaded with `/skillname`:

| Skill | Description |
|-------|-------------|
| `/web` | Web project stack (TanStack, shadcn/ui, TailwindCSS, etc.), state management philosophy, code principles, TypeScript mantras, and expert perspectives to channel |
| `/react` | Advanced React patterns cheat-sheet — re-renders, memoization, reconciliation, refs, portals, data fetching, error handling, performance (distilled from "Advanced React" by Nadia Makarevich) |

### Shell aliases (`~/.zshrc`)
- **`cc`** — Launch Claude Code in a tmux session (see [Setup](#7-shell-aliases))
- **`pr-review`** — Review the current branch's PR using `claude -p` in a fresh context, free from the coding session's blind spots

## Remote Workflow

That’s pretty cool for validating permissions and iterating from the mobile device.

<img width="395" height="836" alt="image" src="https://github.com/user-attachments/assets/8d5f788a-0a31-4235-97fc-f779580ec782" />



This config supports a mobile workflow:
1. Run Claude Code inside **tmux** on your Mac
2. Receive push notifications on iPhone/iPad via **ntfy.sh**
3. SSH in from **Termius** to respond (locally via `hostname.local` or remotely via **Tailscale**)

## Setup

### 1. Clone config

```bash
# Back up existing config first
cp -r ~/.claude ~/.claude-backup

# Clone
git clone git@github.com:loiclaudet/claude-config.git ~/.claude
```

### 2. ntfy (push notifications)

1. Install [ntfy](https://apps.apple.com/app/ntfy/id1625396347) on iPhone/iPad
2. Subscribe to a private topic (e.g. `my-claude-xyz123`)
3. Update the topic name in `settings.json` (replace `lodz-claude-x7k9m2`)

### 3. tmux (persistent sessions)

```bash
# macOS
brew install tmux
```

Enable mouse scrolling — add to `~/.tmux.conf`:

```conf
set -g mouse on
```

### 4. Tailscale (remote access)

1. Install [Tailscale](https://tailscale.com/download) on Mac and iPhone/iPad
2. Sign in with the same account on both devices
3. Get your Mac's Tailscale IP: `tailscale ip -4`

### 5. Enable SSH on Mac

1. **System Settings → General → Sharing → Remote Login** → On
2. Ensure your user is in the allowed users list

### 6. Termius (SSH from mobile)

1. Install [Termius](https://apps.apple.com/app/termius/id549039908) on iPhone/iPad
2. Add host:
   - **Hostname**: your Tailscale IP (e.g. `100.x.x.x`) or `your-mac.local` for local network
   - **Username**: your macOS username
   - **Password**: your macOS login password

### 7. Shell aliases

Add this to your `~/.zshrc`:

```bash
# Claude Code in tmux
unalias cc 2>/dev/null
cc() {
  local session="${1:-claude}"
  if tmux has-session -t "$session" 2>/dev/null; then
    tmux attach -t "$session"
  else
    if [ -z "$1" ]; then
      local i=2
      while tmux has-session -t "claude$i" 2>/dev/null; do
        ((i++))
      done
      if tmux has-session -t "claude" 2>/dev/null; then
        session="claude$i"
      fi
    fi
    tmux new -s "$session" -d
    tmux send-keys -t "$session" "claude -r" Enter
    tmux attach -t "$session"
  fi
}
```

**Usage:**

| Command | Behavior |
|---------|----------|
| `cc` | Attach to existing "claude" session, or create one |
| `cc` | If "claude" is taken, creates "claude2", "claude3", etc. |
| `cc work` | Create/attach to a named session |

#### `pr-review` — PR code review with fresh context

Inspired by [Boris Cherny's recommendation](https://x.com/bcherny) to review Claude-written code using `claude -p` in a **fresh context window**, so Claude doesn't have blind spots from the same session that wrote the code.

Add this to your `~/.zshrc`:

```bash
pr-review() {
  local pr_number
  pr_number=$(gh pr view --json number -q '.number' 2>/dev/null)
  if [ -z "$pr_number" ]; then
    echo "No PR found for current branch. Push and create a PR first."
    return 1
  fi
  local diff
  diff=$(gh pr diff "$pr_number")
  echo "$diff" | claude -p \
    "You are reviewing a pull request. Here is the diff. Review it for:
- Bugs and logic errors
- Security vulnerabilities
- Dead code and unnecessary complexity
- Over-engineering
- Missing error handling at system boundaries
- Code that doesn't match surrounding conventions

Be concise. Only flag real issues, not style nitpicks. For each issue, reference the file and line."
}
```

**Usage:** From any branch with an open PR, run `pr-review`.

**Prerequisites:** `gh` CLI and `claude` CLI installed and authenticated.

### Workflow

1. Start Claude: `cc`
2. Walk away — when Claude needs you, your phone buzzes
3. Open Termius → SSH in → `tmux attach -t claude`
4. Respond, then detach: `Ctrl+b d`
