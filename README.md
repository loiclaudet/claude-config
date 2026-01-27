# Claude Code Config

My [Claude Code](https://docs.anthropic.com/en/docs/claude-code) configuration — settings, hooks, and skills.

## What's Inside

### `settings.json`
- **Notification hooks**: Sound alerts + [ntfy.sh](https://ntfy.sh) push notifications for permission prompts and idle state (10s delay, auto-cancelled if responded to)
- **Hook cancellation**: `PreToolUse` and `UserPromptSubmit` hooks that kill pending notifications when you respond in time

### `CLAUDE.md`
Global preferences applied to all projects.

### `skills/`
On-demand guidelines loaded with `/skillname`:

| Skill | Description |
|-------|-------------|
| `/web` | Web project stack (TanStack, shadcn/ui, TailwindCSS, etc.), state management philosophy, code principles, TypeScript mantras, and expert perspectives to channel |
| `/react` | Advanced React patterns cheat-sheet — re-renders, memoization, reconciliation, refs, portals, data fetching, error handling, performance (distilled from "Advanced React" by Nadia Makarevich) |

## Remote Workflow

This config supports a mobile workflow:
1. Run Claude Code inside **tmux** on your Mac
2. Receive push notifications on iPhone/iPad via **ntfy.sh**
3. SSH in from **Termius** to respond (locally via `hostname.local` or remotely via **Tailscale**)

## Usage

```bash
# Clone into your Claude config directory
git clone git@github.com:loiclaudet/claude-config.git ~/.claude
```

> **Note**: Back up your existing `~/.claude` directory first if you have one.
