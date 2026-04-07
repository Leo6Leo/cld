# cld

Interactive TUI to browse and resume Claude Code conversations across all projects on your machine.

Unlike the built-in `/resume` command which only shows conversations for the current directory, `cld` shows **all** projects and conversations.

Install via Homebrew:

```sh
brew install Leo6Leo/tap/cld
```

## Features

- **Two view modes**: Projects (grouped by folder) and Recent (flat list by time)
- **Instant filter**: Search across project names and conversation content
- **Resume sessions**: Select a conversation to resume it via `claude --resume`
- **Auto cd**: Terminal working directory changes to the project folder
- **Remembers preferences**: View mode is persisted between sessions

## Install

1. Copy the script somewhere on your PATH:

```sh
cp cld /usr/local/bin/
```

Or add a shell function to your `~/.zshrc` (recommended, enables cd-after-exit):

```sh
cld() {
  local tmpfile=$(mktemp)
  CLAUDE_PROJECTS_CD_FILE="$tmpfile" /path/to/cld "$@"
  if [[ -s "$tmpfile" ]]; then
    local dir=$(cat "$tmpfile")
    cd "$dir" 2>/dev/null
  fi
  rm -f "$tmpfile"
}
```

2. Run it:

```sh
cld
```

## Usage

### Interactive (default)

```sh
cld
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
cld --list              # Table view
cld --list -c           # With conversation details
cld --list -r           # Recent conversations (flat)
cld --list TERM         # Filter by keyword
```

## Requirements

- Python 3.6+
- Claude Code CLI (`claude`)

## How it works

Claude Code stores conversations as JSONL files in `~/.claude/projects/`, organized by project directory path. This tool scans those files, extracts metadata (first message, session name, timestamps), and presents them in a navigable TUI. When you select a conversation, it `cd`s into the project directory and runs `claude --resume <session-id>`.
