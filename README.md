# claude-projects

Interactive TUI to browse and resume Claude Code conversations across all projects on your machine.

Unlike the built-in `/resume` command which only shows conversations for the current directory, `claude-projects` shows **all** projects and conversations.

## Features

- **Two view modes**: Projects (grouped by folder) and Recent (flat list by time)
- **Instant filter**: Search across project names and conversation content
- **Resume sessions**: Select a conversation to resume it via `claude --resume`
- **Auto cd**: Terminal working directory changes to the project folder
- **Remembers preferences**: View mode is persisted between sessions

## Install

1. Copy the script somewhere on your PATH:

```sh
cp claude-projects /usr/local/bin/
```

Or add a shell function to your `~/.zshrc` (recommended, enables cd-after-exit):

```sh
claude-projects() {
  local tmpfile=$(mktemp)
  CLAUDE_PROJECTS_CD_FILE="$tmpfile" /path/to/claude-projects "$@"
  if [[ -s "$tmpfile" ]]; then
    local dir=$(cat "$tmpfile")
    cd "$dir" 2>/dev/null
  fi
  rm -f "$tmpfile"
}
```

2. Run it:

```sh
claude-projects
```

## Usage

### Interactive (default)

```sh
claude-projects
```

| Key | Action |
|---|---|
| `↑/↓` or `j/k` | Navigate |
| `Enter` | Expand project / Resume conversation |
| `Space` | Expand project / Show session ID |
| `/` | Start filtering |
| `Esc` | Exit filter → Clear filter → Quit |
| `1` / `2` / `Tab` | Switch view mode |
| `g` / `G` | Jump to top / bottom |
| `PgUp` / `PgDn` | Page up / down |
| `q` | Quit |

### Non-interactive

```sh
claude-projects --list              # Table view
claude-projects --list -c           # With conversation details
claude-projects --list -r           # Recent conversations (flat)
claude-projects --list TERM         # Filter by keyword
```

## Requirements

- Python 3.6+
- Claude Code CLI (`claude`)

## How it works

Claude Code stores conversations as JSONL files in `~/.claude/projects/`, organized by project directory path. This tool scans those files, extracts metadata (first message, session name, timestamps), and presents them in a navigable TUI. When you select a conversation, it `cd`s into the project directory and runs `claude --resume <session-id>`.
