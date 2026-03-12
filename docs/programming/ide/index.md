---
title: IDE
icon: simple/zedindustries
---

## General

Local exlude from git: `.git/info/exclude`, great for ignoring IDE specific files

## Zed

- fast: written in Rust
- minimalistic: no distraction, no bambilion of plugins needed

### Shorcuts

| Shotcut | Action | Default| Note |
|---------|--------|--------|-------|
| `Ctrl+Shift+P` | Open command palette | ✓ ||
| `Ctrl+S` | Save file | ✓ ||
| `Ctrl+Z` | Undo last change | ✓ ||
| `Ctrl+J` | Show/Hide terminal | ✓ ||
| `Ctrl+B` | Show/Hide left dock | ✓ ||
| `Ctrl+Alt+B` | Show/Hide right dock | ✓ ||
| `Ctrl+Shift+E` | Project panel (tree) | ✓ ||
| `Ctrl+Shift+G` | Git panel | ✓ ||

### Configuration tips

Local config files are in `project_root/.zed/` and global config is in `~/.config/zed/`.
Typical files: `settings.json` for general settings, `keybindings.json` for shortcuts, `snippets.json` for code snippets, `extensions.json` for extensions, `tasks.json` for tasks, `launch.json` for launch configurations.

1) Default terminal to fish (`settings.json`):

  ```json
  "terminal": {
    "shell": {
      "program": "fish",
    },
  },
  ```

2) Hide dirs like `__pycache__` and `.venv` in the project tree (`settings.json`):

  ```json
  "file_scan_exclusions": [
    "**/.git",
    "**/.svn",
    "**/.hg",
    "**/CVS",
    "**/.DS_Store",
    "**/Thumbs.db",
    "**/.classpath",
    "**/.settings",
    "**/__pycache__",
    "**/.venv",
    "**/.idea",
  ]
  ```
3) Visuals (`settings.json`)

  ```json
  "ui_font_size": 16,
  "buffer_font_size": 14.0,
  "theme": {
    "mode": "system",
    "light": "One Light",
    "dark": "One Dark Pro",
  },  
  ```
4) Specific formater, e.g.u Jinja (`settings.json`)
  ```json
  {
    "file_types": {
      "Jinja2": ["html"],
    },
    "languages": {
      "Jinja2": {
        "formatter": {
          "external": {
            "command": "uv",
            "arguments": [
              "run",
              "djlint",
              "-",
              "--reformat",
              "--profile=jinja",
              "--preserve-blank-lines",
            ],
          },
        },
      },
    },
  }
  ```
5) [Tasks](https://zed.dev/docs/tasks) (`tasks.json`)

  ```json
  [
    {
      "label": "1 Lint",
      "command": "uv run ruff format . && uv run ruff check .",
      "use_new_terminal": false,
      "allow_concurrent_runs": false,
      "reveal": "always",
      "hide": "never",
      "shell": {
        "program": "sh",
      },
      "show_summary": true,
      "show_command": true,
      "tags": ["ci", "ruff"],
    },
  ]
  ```


### PyCharm

When you have package / src layout, you need to mark `src` as "Sources Root" in PyCharm.
