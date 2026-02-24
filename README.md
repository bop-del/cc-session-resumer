# cc-session-resumer

A fast, keyboard-driven terminal picker for all your Claude Code sessions.
Browse every session across all projects, preview details and your past prompts,
and open any session in a new terminal window — without leaving the terminal.

![demo](https://raw.githubusercontent.com/bop-del/cc-session-resumer/main/demo.png)

## What it does

- Aggregates all Claude Code sessions from `~/.claude/projects/*/`
  (both indexed `sessions-index.json` entries and raw `.jsonl` fallbacks)
- Shows a sorted list (newest first) with crash indicators, message counts,
  project names, and summaries
- Right-side preview pane shows: session title, project/path, token usage,
  model used, top tools, and your last N prompts with timestamps
- Opens selected sessions in a new terminal window (Ghostty by default, configurable)
- Stays open after each selection so you can launch multiple sessions in a row

## Design

Three scripts, each with a single responsibility:

| Script | Role |
|--------|------|
| `cc-session-resumer` | Main entry point — loads sessions, builds fzf picker, handles key bindings |
| `cc-session-resumer-preview` | fzf preview pane — renders session card from `.jsonl` data |
| `cc-session-resumer-open` | Session opener — builds resume command, opens in configured terminal (default: Ghostty) |

`~/.local/bin/cc-session-resumer`, `cc-session-resumer-preview`, and `cc-session-resumer-open` are symlinks into this repo.
Edit here → changes take effect immediately, no install step needed.

### Key bindings (vim mode, default)

| Key | Action |
|-----|--------|
| `j` | Move down |
| `k` | Move up |
| `l` | Open selected session in new terminal window |
| `ESC` | Quit |

Use `cc-session-resumer --normal` for multi-select mode (TAB + ENTER).

### How sessions are loaded

Claude Code stores sessions in `~/.claude/projects/<encoded-path>/`.
Each project may have a `sessions-index.json` (structured metadata) and/or
raw `.jsonl` files (one JSON object per line). `cc-session-resumer` reads both, deduplicates
by session ID, and sorts by modification time (newest first).

**Crash detection:** a session is flagged with `!` if its last conversation
message was from the user (i.e. Claude never responded to the final prompt).

### Preview pane

The right-side preview pane (`cc-session-resumer-preview`) reads the raw `.jsonl` for the
selected session and renders:

- Bold cyan title (first prompt / summary), up to 2 lines
- Project name + truncated path + `[CRASHED]` tag (right-aligned) if crashed
- Message count, session duration, created/modified dates
- Full session ID + ready-to-paste `claude --resume <id>` command
- Token usage (input/output/cache), model name, top tools used
- Last 8 user prompts (newest first) with `HH:MM` timestamps

## Prerequisites

| Requirement | Install |
|-------------|---------|
| Python 3.9+ | Ships with macOS, or `brew install python` |
| fzf 0.50+   | `brew install fzf` |
| Claude Code | `npm install -g @anthropic-ai/claude-code` |
| Ghostty terminal (default) | https://ghostty.org |

> **Warp users:** Set `export CC_SESSION_RESUMER_TERMINAL=warp` in your `.zshrc`.
> Warp also requires macOS Accessibility: System Settings → Privacy & Security → Accessibility → Warp ✓

## Install

```bash
# 1. Clone
git clone https://github.com/bop-del/cc-session-resumer.git ~/code/cc-session-resumer

# 2. Symlink into PATH
mkdir -p ~/.local/bin
ln -s ~/code/cc-session-resumer/cc-session-resumer ~/.local/bin/cc-session-resumer
ln -s ~/code/cc-session-resumer/cc-session-resumer-preview ~/.local/bin/cc-session-resumer-preview
ln -s ~/code/cc-session-resumer/cc-session-resumer-open ~/.local/bin/cc-session-resumer-open

# 3. Ensure ~/.local/bin is on your PATH (add to ~/.zshrc if needed)
export PATH="$HOME/.local/bin:$PATH"
```

## Usage

```bash
cc-session-resumer          # open picker (vim mode, default)
cc-session-resumer --normal # multi-select mode (TAB to select multiple, ENTER to open all)
```


## Terminal Support

By default, sessions open in **Ghostty**. Set `CC_SESSION_RESUMER_TERMINAL` to switch:

| Value | Behavior |
|-------|----------|
| `ghostty` (default) | Opens new Ghostty window via `ghostty -- zsh -c` |
| `warp` | Opens new Warp tab via AppleScript |
| anything else | Copies command to clipboard, prints to stdout |

```sh
# Add to ~/.zshrc (optional — ghostty is the default)
export CC_SESSION_RESUMER_TERMINAL=ghostty
```

## Future features

- **Search / fuzzy filter** — press `/` to filter sessions by project, summary, or session ID
- **Date filter** — show only sessions from the last N days
- **Session count** — display total count in the header line

## Files

```
cc-session-resumer/
├── cc-session-resumer          # main picker (fzf launcher + session loader)
├── cc-session-resumer-preview  # fzf preview pane renderer
├── cc-session-resumer-open     # terminal opener — dispatches to ghostty/warp/fallback (called by fzf execute-silent)
└── README.md
```
