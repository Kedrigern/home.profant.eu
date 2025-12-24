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

Set padding to zero, because zellij has own padding, `~/.config/ghostty/config`:

```
window-padding-x = 0
window-padding-y = 0
```

Set `fish` as default shell for zellij, in `~/.config/zellij/config.kdl`:

```kdl
default_shell "fish"
copy_on_select true
```

Aliases and defautl editor for fish, in `~/.config/fish/config.fish`:

```fish
set fish_greeting       # disable greeting lines
set -gx EDITOR nvim
  
if status is-interactive
    alias z="zellij"
    alias za="zellij attach"
end
```

Set the promt to be minimalistic if we are in zellij, edit `~/.config/fish/functions/fish_prompt.fish`:

```fish
function fish_prompt --description 'Write out the prompt'
    if set -q ZELLIJ
        echo -n -s (set_color green) '$ ' (set_color normal)
        return
    end
    set -l last_pipestatus $pipestatus
    set -lx __fish_last_status $status # Export for __fish_print_pipestatus.
    set -l normal (set_color normal)
    set -q fish_color_status
    or set -g fish_color_status red

    # Color the prompt differently when we're root
    set -l color_cwd $fish_color_cwd
    set -l suffix '>'
    if functions -q fish_is_root_user; and fish_is_root_user
        if set -q fish_color_cwd_root
            set color_cwd $fish_color_cwd_root
        end
        set suffix '#'
    end

    # Write pipestatus
    # If the status was carried over (if no command is issued or if `set` leaves the status untouched), don't bold it.
    set -l bold_flag --bold
    set -q __fish_prompt_status_generation; or set -g __fish_prompt_status_generation $status_generation
    if test $__fish_prompt_status_generation = $status_generation
        set bold_flag
    end
    set __fish_prompt_status_generation $status_generation
    set -l status_color (set_color $fish_color_status)
    set -l statusb_color (set_color $bold_flag $fish_color_status)
    set -l prompt_status (__fish_print_pipestatus "[" "]" "|" "$status_color" "$statusb_color" $last_pipestatus)

    echo -n -s (prompt_login)' ' (set_color $color_cwd) (prompt_pwd) $normal (fish_vcs_prompt) $normal " "$prompt_status $suffix " "
end
```

Title of window or zellij pane, edit `~/.config/fish/functions/fish_title.fish`:

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
  
## Zellij

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

| Akce	| Příkaz (Fish) |	Alias (pokud máte) |
|--------------------|------------------|----|
|Start (pojmenovaná) | zellij -s <name> | – |
|Start (random)	     | zellij           | z |
|Seznam běžících	   | zellij ls	      | – |
|Připojit (menu)	   | zellij attach    |	za |
|Připojit (jméno)	   | zellij a <name>	| za <name> |
|Zrušit (zvenku)	   | zellij d <name>	| – |
|Odpojit (zevnitř)	  |Ctrl + o -> d    |	– |
