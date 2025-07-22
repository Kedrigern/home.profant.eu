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

# Heredocs
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


```fish
fish_config browse         # config

(cmd)                      # capture return value of cmd

set my_var "Hello"
set my_list 1 2 3          # Every var is posible list
set -q var                 # var exists?

# Arithmetic
math $i + 1
math cos 2 x pi
math '(5 + 2) * 4'
```