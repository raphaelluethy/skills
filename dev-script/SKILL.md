---
name: dev-script
description: Generate a dev.sh script that uses tmux to manage development services (dev server, database, etc.) with flags for clean, restart, down, and db-only modes.
argument-hint: "[project-name]"
disable-model-invocation: true
---

# Dev Script Generator

Generate a `dev.sh` bash script for the current project. The script uses **tmux** to manage development services in named sessions with split panes.

## Required Input

Ask the user for the following if not provided via `$ARGUMENTS` or context:

1. **Session/project name** — used as the tmux session name (e.g., `myapp`)
2. **Services to run** — what commands run in each pane (e.g., `bun dev`, `docker compose up`, `cargo watch`)
3. **Package manager** — for the `--clean` flag reinstall step (e.g., `bun`, `npm`, `pnpm`)
4. **Files to remove on clean** — lock files and dependency dirs to delete (e.g., `node_modules bun.lockb`)

## Script Structure

The generated `dev.sh` must follow this exact pattern:

```bash
#!/bin/bash
```

### 1. Argument parsing

Support these flags via a `for arg in "$@"` loop with `case`:

| Flag        | Variable    | Purpose                                      |
|-------------|-------------|----------------------------------------------|
| `--clean`   | `CLEAN`     | Full teardown: stop services, delete deps, reinstall, then start |
| `--db`      | `DB_ONLY`   | Start only the database/infrastructure service |
| `--restart` | `RESTART`   | Kill existing session and recreate            |
| `--down`    | `DOWN`      | Shut everything down and exit                 |

All default to `false`.

### 2. Session existence check

```bash
SESSION_EXISTS=false
if tmux has-session -t <name> 2>/dev/null; then
    SESSION_EXISTS=true
fi
```

### 3. Mode handlers (in this order)

**--down**: Kill tmux session if exists, stop infrastructure (e.g., `docker compose down`), then `exit 0`.

**--clean**: Kill session, stop infrastructure with `-v`, remove dependency files, reinstall, `sleep 1`, set `SESSION_EXISTS=false`. Does NOT exit — falls through to start.

**--db**: Kill existing session, create new session with only the infrastructure service, attach, `exit 0`.

**--restart / session exists**: Kill existing session, fall through to creation.

### 4. Session creation

- Create a detached tmux session with the first service in the first pane
- Split the window vertically for each additional service
- Select the first pane after setup
- Print what was created

### 5. Attach

```bash
tmux attach -t <name>
```

## Output

Write the script to `dev.sh` in the project root and make it executable with `chmod +x dev.sh`.

