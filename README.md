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
- A **layout** defines how panes are split using a binary tree:
  - `split: vertical` creates a left/right split
  - `split: horizontal` creates a top/bottom split
  - Leaf panes can be left empty (null) to inherit the window's `dir`, or specify their own `dir` to override it.

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

If you're looking for something more featureful — like automatic session saving/restoring, persisting pane contents, or surviving server restarts — take a look at [tmux-resurrect](https://github.com/tmux-plugins/tmux-resurrect).

## Attribution

Authored by @claude, guided by @pmccarren
