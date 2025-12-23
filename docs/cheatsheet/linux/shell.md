# Shell


## Comparasion

|  Bash | Fish | Note |
|--------------------------|--------------|--------------------------------|
| ``` `cmd` ```, `$(cmd)`  |      `(cmd)` | Capture return value of cmd    |
| `var=val` | `set var val` | Assign variable |
| `for i in $(seq 1 10); do echo "$i"; done` | `for i in (seq 1 10); echo $i; end` | For cycle |


## Bash

```bash
for i in $(seq 1 10);
do
    echo $i
done

cat <<EOF      
some string
some more string
EOF
```

## Fish

Native support for multiline commands.

`ctrl+r` search in history  
`ctrl+c` escape from multiline command (access to the history)  
`ctrl+u` delete all from cursor to the start of line  
`ctrl+k` delete all from cursor to the end of line  
`ctrl+a` cursor to the start

Multiline string in fish:  
`printf %s\n "some string" "some more string"`

`~/.config/fish/config.fish`

```fish
fish_config browse          # config
fish_add_path ~/.cargo/bin/ # add to PATH

(cmd)                      # capture return value of cmd

set my_var "Hello"
set -q var                 # var exists?

# Arithmetic
math $i + 1
math cos 2 x pi
math '(5 + 2) * 4'
```

## Ghostty + Fish + Zellij

```fish
sudo dnf install ghostty fish zellij wl-clipboard git
```

In `~/.config/ghostty/config`:

```
window-padding-x = 0
window-padding-y = 0
```

In `~/.config/zellij/config.kdl`:

```
default_shell "fish"
copy_on_select true
```

In `~/.config/fish/config.fish`:

```fish
if status is-interactive
    alias z="zellij"
    alias za="zellij attach"
end
```

In `~/.config/fish/functions/fish_prompt.fish`:

```fish
function fish_prompt
    # Pokud jsme v Zellij, zobraz jen zelený dolar
    if set -q ZELLIJ
        echo -n -s (set_color green) '$ ' (set_color normal)
        return
    end

    # ... ZDE by následoval defaultní kód pro prompt mimo Zellij ...
    # (Pokud soubor vytvoříte nový, Fish použije jen to nahoře. 
    #  Chcete-li zachovat systémový prompt mimo Zellij, zkopírujte si 
    #  nejdřív obsah: cp /usr/share/fish/functions/fish_prompt.fish ~/.config/fish/functions/)
end
```

In `~/.config/fish/functions/fish_title.fish`:

```fish
function fish_title
    # 1. SSH
    set -l ssh_part ""
    if set -q SSH_TTY
        set -l host_name (prompt_hostname | string sub -l 10)
        set ssh_part "[$host_name] "
    end

    # 2. Cesta
    set -l path_part (prompt_pwd)

    # 3. Git
    set -l git_part ""
    if git rev-parse --is-inside-work-tree >/dev/null 2>&1
        set -l branch_name (git branch --show-current 2>/dev/null | string trim)
        if test -n "$branch_name"
            set git_part " ($branch_name)"
        end
    end

    # 4. Command (skrýt 'fish')
    set -l cmd_part ""
    set -l current_cmd $argv[1]
    if test -z "$current_cmd"; set current_cmd (status current-command); end
    if test "$current_cmd" != "fish"; set cmd_part " $current_cmd"; end

    echo -n -s $ssh_part $path_part $git_part $cmd_part
end
```
  
Cheat sheet:

- Spuštění: `z` (nová session), `za` (připojit k běžící).

Kopírování/Vložení:

- `Shift + Prostřední myš` = Vložit (obejde Zellij).
- `Shift + Výběr myší` = Kopírovat i přes více panelů (obejde Zellij).
- `Ctrl + Shift + V` = Vložit ze schránky.

Zellij klávesy (výchozí):

- `Ctrl + g` = Zamknout/Odemknout ovládací panel.
- `Ctrl + p + n` = Nový panel.
- `Ctrl + t + n` = Nový tab.
- `Alt + šipky` = Pohyb mezi panely (často funguje i bez prefixu, záleží na configu).
