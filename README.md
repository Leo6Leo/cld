# cld

Interactive TUI to browse and resume Claude Code conversations across all projects on your machine.

Unlike the built-in `/resume` command which only shows conversations for the current directory, `cld` shows **all** projects and conversations.

## Demo

### Projects View

```
 [1] Projects   [2] Recent
+-----------------------------------------------------------------------+
| > ~/projects/webapp           (12 convos, 45MB, 2h ago)               |
| v ~/projects/api-server       (8 convos, 120MB, 5h ago)               |
|     -- 5h ago    1MB  how do I add rate limiting to the auth endpoi.. |
|     -- 5h ago   95MB  fix the failing integration tests in ci         |
|     -- 5d ago    2KB  (command)                                       |
| > ~/work/mobile-app           (4 convos, 5MB, 3h ago)                 |
| > ~/projects/docs-site        (9 convos, 359KB, 2d ago)               |
| > ~/work/infra                (1 convo, 68MB, 6d ago)                 |
|                                                                       |
| 5 projects, 34 conversations                                          |
| up/down:move  Enter:open  /:filter  1/2:view  Esc/q:quit              |
+-----------------------------------------------------------------------+
```

### Recent View

```
 [1] Projects   [2] Recent
+-----------------------------------------------------------------------+
|    1h ago   45KB  ~/projects/webapp       add dark mode toggle to ..  |
|    3h ago  220KB  ~/work/mobile-app       (command)                   |
|    5h ago    1MB  ~/projects/api-server   how do I add rate limit ..  |
|    5h ago   95MB  ~/projects/api-server   fix the failing integrat..  |
|    2d ago  321KB  ~/projects/docs-site    (command)                   |
|    5d ago    2KB  ~/projects/api-server   (command)                   |
|    6d ago   68MB  ~/work/infra            set up terraform for sta..  |
|                                                                       |
| 34 conversations                                                      |
| up/down:move  Enter:open  /:filter  1/2:view  Esc/q:quit              |
+-----------------------------------------------------------------------+
```

### Filter

```
 [1] Projects   [2] Recent
 /api_
+-----------------------------------------------------------------------+
| > ~/projects/api-server       (8 convos, 120MB, 5h ago)               |
| > ~/projects/webapp           (12 convos, 45MB, 2h ago)               |
|                                                                       |
| 2 projects, 20 conversations                                          |
| Type to filter | Enter:confirm  Esc:exit  Backspace:delete            |
+-----------------------------------------------------------------------+
```

## Install

### Homebrew (recommended)

```sh
brew tap Leo6Leo/cld https://github.com/Leo6Leo/cld
brew install cld
```

### Manual

Copy the script somewhere on your PATH:

```sh
cp cld /usr/local/bin/
```

### Shell wrapper (enables cd-after-exit)

Add to your `~/.zshrc` or `~/.bashrc`:

```sh
cld() {
  local tmpfile=$(mktemp)
  CLAUDE_PROJECTS_CD_FILE="$tmpfile" command cld "$@"
  if [[ -s "$tmpfile" ]]; then
    local dir=$(cat "$tmpfile")
    cd "$dir" 2>/dev/null
  fi
  rm -f "$tmpfile"
}
```

This ensures your terminal stays in the project directory after resuming a conversation.

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

## Features

- **Two view modes**: Projects (grouped by folder) and Recent (flat list by time)
- **Instant filter**: Search across project names and conversation content
- **Resume sessions**: Select a conversation and press Enter to resume via `claude --resume`
- **Auto cd**: Terminal working directory changes to the project folder
- **Remembers preferences**: View mode is persisted between sessions
- **Fast startup**: Only reads first 50 lines of each conversation file, handles 100MB+ files without lag

## Requirements

- Python 3.6+
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI

## How it works

Claude Code stores conversations as JSONL files in `~/.claude/projects/`, organized by project directory path. `cld` scans those files, extracts metadata (first message, session name, timestamps), and presents them in a navigable TUI built with Python's `curses`. When you select a conversation, it `cd`s into the project directory and runs `claude --resume <session-id>`.
