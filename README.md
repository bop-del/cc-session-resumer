# cc-session-resumer

A fast, keyboard-driven terminal picker for all your Claude Code sessions.
Browse every session across all projects, preview details and your past prompts,
and open any session in a new Warp tab — without leaving the terminal.

![demo](https://raw.githubusercontent.com/bop-del/cc-session-resumer/main/demo.png)

## What it does

- Aggregates all Claude Code sessions from `~/.claude/projects/*/`
  (both indexed `sessions-index.json` entries and raw `.jsonl` fallbacks)
- Shows a sorted list (newest first) with crash indicators, message counts,
  project names, and summaries
- Right-side preview pane shows: session title, project/path, token usage,
  model used, top tools, and your last N prompts with timestamps
- Opens selected sessions in a new Warp terminal tab via AppleScript
- Stays open after each selection so you can launch multiple sessions in a row

## Design

Three scripts, each with a single responsibility:

| Script | Role |
|--------|------|
| `ccs` | Main entry point — loads sessions, builds fzf picker, handles key bindings |
| `ccs-preview` | fzf preview pane — renders session card from `.jsonl` data |
| `ccs-open` | Session opener — builds resume command, copies to clipboard, opens Warp tab |

`~/.local/bin/ccs`, `ccs-preview`, and `ccs-open` are symlinks into this repo.
Edit here → changes take effect immediately, no install step needed.

### Key bindings (vim mode, default)

| Key | Action |
|-----|--------|
| `j` | Move down |
| `k` | Move up |
| `l` | Open selected session in new Warp tab |
| `/` | Enter fuzzy search mode |
| `ESC` | Quit |

Use `ccs --normal` for multi-select mode (TAB + ENTER).

### How sessions are loaded

Claude Code stores sessions in `~/.claude/projects/<encoded-path>/`.
Each project may have a `sessions-index.json` (structured metadata) and/or
raw `.jsonl` files (one JSON object per line). `ccs` reads both, deduplicates
by session ID, and sorts by modification time (newest first).

**Crash detection:** a session is flagged with `!` if its last conversation
message was from the user (i.e. Claude never responded to the final prompt).

### Preview pane

The right-side preview pane (`ccs-preview`) reads the raw `.jsonl` for the
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
| Warp terminal | https://warp.dev |
| macOS Accessibility for Warp | System Settings → Privacy & Security → Accessibility |

> **Accessibility permission required:** `ccs-open` uses AppleScript to open
> a new Warp tab. Enable it in:
> **System Settings → Privacy & Security → Accessibility → Warp ✓**

## Install

```bash
# 1. Clone
git clone https://github.com/bop-del/cc-session-resumer.git ~/code/cc-session-resumer

# 2. Symlink into PATH
mkdir -p ~/.local/bin
ln -s ~/code/cc-session-resumer/ccs ~/.local/bin/ccs
ln -s ~/code/cc-session-resumer/ccs-preview ~/.local/bin/ccs-preview
ln -s ~/code/cc-session-resumer/ccs-open ~/.local/bin/ccs-open

# 3. Ensure ~/.local/bin is on your PATH (add to ~/.zshrc if needed)
export PATH="$HOME/.local/bin:$PATH"
```

## Usage

```bash
ccs          # open picker (vim mode, default)
ccs --normal # multi-select mode (TAB to select multiple, ENTER to open all)
```

## Files

```
cc-session-resumer/
├── ccs           # main picker (fzf launcher + session loader)
├── ccs-preview   # fzf preview pane renderer
├── ccs-open      # Warp tab opener (called by fzf execute-silent)
└── README.md
```
