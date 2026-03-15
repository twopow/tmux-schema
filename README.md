# tmux-schema

Bootstrap tmux sessions from a YAML file. Define your sessions, windows, pane splits, and working directories in one place, then run a single command to set everything up.

## Usage

```
tmux-schema [--file <path>] [--only <name>] [--attach] [--dry-run] [--help]
```

| Flag | Description |
|------|-------------|
| `--file <path>` | Schema file (default: `./tmux-schema.yaml`, then `~/.tmux-schema.yaml`) |
| `--only <name>` | Only create the named session (repeatable) |
| `--attach` | Force attach even if sessions already existed |
| `--dry-run` | Print tmux commands without executing |

## Schema format

Create a `tmux-schema.yaml`:

```yaml
sessions:
  - name: work
    windows:
      - name: projects
        dir: ~/projects
      - name: split-view
        dir: ~/projects
        layout:
          split: vertical
          left:
          right:
            split: horizontal
            top:
              cmd: git status
            bottom:

  - name: play
    windows:
      - name: play
        dir: ~/play
        layout:
          split: vertical
          left:
          right:
            dir: ~/play/sandbox
```

### How it works

- Each **window** has a `dir` — the default working directory for all its panes.
- Each **window** can have a `cmd` — a command to run in each pane after it's created (as if you typed it in).
- A **layout** defines how panes are split using a binary tree:
  - `split: vertical` creates a left/right split
  - `split: horizontal` creates a top/bottom split
  - Leaf panes can be left empty (null) to inherit the window's `dir` and `cmd`, or specify their own to override.

### Example: three-pane layout

```
+-------------------+-------------------+
|                   |                   |
|                   |       top         |
|                   |                   |
|       left        +-------------------+
|                   |                   |
|                   |      bottom       |
|                   |                   |
+-------------------+-------------------+
```

```yaml
layout:
  split: vertical
  left:
  right:
    split: horizontal
    top:
    bottom:
```

### Example: running commands in panes

```yaml
- name: dev
  dir: ~/projects/myapp
  cmd: git status
  layout:
    split: vertical
    left:
    right:
      cmd: npm run dev
```

The left pane inherits both `dir` and `cmd` from the window — it opens in `~/projects/myapp` and runs `git status`. The right pane overrides `cmd` to run `npm run dev` instead, but still inherits the directory.

## Requirements

- bash
- [tmux](https://github.com/tmux/tmux)
- [yq](https://github.com/mikefarah/yq) (YAML processor)

## Install

```sh
# Copy the script somewhere on your PATH
curl -o ~/.local/bin/tmux-schema https://raw.githubusercontent.com/twopow/tmux-schema/main/tmux-schema
chmod +x ~/.local/bin/tmux-schema
```

Or clone and symlink:

```sh
git clone https://github.com/twopow/tmux-schema.git
ln -s "$(pwd)/tmux-schema/tmux-schema" ~/.local/bin/tmux-schema
```

## Quick start

```sh
# Preview what would happen
tmux-schema --dry-run

# Create all sessions
tmux-schema

# Create only a specific session
tmux-schema --only work
```

## Alternatives

tmux-schema is intentionally minimal — a single bash script, a simple YAML file, no plugins. It's a good fit if you want a declarative way to spin up a known set of sessions and don't need much else.

If you're looking for something more featureful:

- [tmuxinator](https://github.com/tmuxinator/tmuxinator) — full-featured session manager with YAML configs, hooks, project templates, and more. Requires Ruby.
- [tmux-resurrect](https://github.com/tmux-plugins/tmux-resurrect) — automatic session saving/restoring, persists pane contents, survives server restarts.

## Attribution

Authored by @claude, guided by @pmccarren
