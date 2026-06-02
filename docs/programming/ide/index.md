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

| Shotcut        | Action               | Default | Note     |
|----------------|----------------------|---------|----------|
| `Ctrl+Shift+P` | Open command palette | ✓ ||
| `Ctrl+S`       | Save file            | ✓ ||
| `Ctrl+P`       | Open file            | ✓ | for example copy file location from GH |
| `Ctrl+Z`       | Undo last change     | ✓ ||
| `Ctrl+J`       | Show/Hide terminal   | ✓ ||
| `Ctrl+B`       | Show/Hide left dock  | ✓ ||
| `Ctrl+Alt+B`   | Show/Hide right dock | ✓ ||
| `Ctrl+Shift+E` | Project panel (tree) | ✓ ||
| `Ctrl+Shift+G` | Git panel            | ✓ ||
| `Alt+click`    | Multicurrsor         | ✓ ||
| `Alt+shift+T`  | Open tasks           | ✓ ||

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
  
6) Turn off autoformating for one format, `settings.json`:

```json
{
  "languages": {
    "YAML": {
      "formatter": "language_server",
    },
  },
  "lsp": {
    "yaml-language-server": {
      "settings": {
        "yaml": {
          "format": {
            "enable": false,
          },
        },
      },
    },
  },
}
```

#### Multi-Profile & Copilot Isolation (Linux)

**The Problem**: GitHub Copilot stores its authentication token globally per OS user inside `~/.config/github-copilot/hosts.json`. Because Zed lacks native multi-profile switching, it will always use this single global identity. Furthermore, browser security features like Multi-Account Containers disrupt desktop application OAuth redirection, resulting in account mismatches.

**The Solution**: XDG Environment Isolation.

To bypass this limitation, you can force Zed into an isolated "sandbox" profile by overriding Linux XDG base directory variables (`XDG_CONFIG_HOME` and `XDG_DATA_HOME`). This forces Zed to create separate config files, extensions, and a distinct Copilot token for each client/project context.

1. Directory Structure Setup: Create dedicated directories for the new profile (e.g., customer-a) to store its independent configuration and data state.
  ```
  mkdir -p ~/.config/zed-customerA
  mkdir -p ~/.local/share/zed-customerA
  ```
2. Icons & Visual Distinction
  To prevent deployment mistakes, use distinct window themes and separate application launcher icons for each profile.
  Default Icon Location: `~/.local/zed.app/share/icons/hicolor/512x512/apps/zed.png`
  Setup Custom Icon: Copy the default icon to the profile folder and alter its hue (via GIMP) or replace it with a colored alternative.
3. CLI Execution & Shell Alias
  To launch the isolated instance from the terminal, inject the environment variables before invoking the binary: `XDG_CONFIG_HOME=~/.config/zed-customerA XDG_DATA_HOME=~/.local/share/zed-customerA ~/.local/zed.app/bin/zed .`
4. GNOME Desktop Integration
  To make the profile searchable via the Super key and pinnable to the GNOME Dash/Dock, create a custom desktop launcher entry.
  Note: `.desktop` files do not support the tilde (~) shortcut. Absolute file paths must be declared.
    ```toml
    [Desktop Entry]
    Type=Application
    Name=Zed (Customer A)
    Comment=Isolated Zed Profile for Customer A Workflow
    Exec=env XDG_CONFIG_HOME=/home/<user>/.config/zed-customera XDG_DATA_HOME=/home/<user>>/.local/share/zed-customera /home/<user>>/.local/zed.app/bin/zed %U
    Icon=/home/<user>/.config/zed-customera/icon.png
    Terminal=false
    Categories=Development;TextEditor;
    StartupNotify=true
    StartupWMClass=zed
    ```
5. How to verify which account: 
  - In ZED: `ctrl+shift+P` -> `zed: open log` -> search `user`
  - From gh copilot config: `sqlite3 ~/.config/zed-<customer>/github-copilot/auth.db "SELECT user_login FROM oauth_tokens;"`



### PyCharm

When you have package / src layout, you need to mark `src` as "Sources Root" in PyCharm to proper autohelp etc.
