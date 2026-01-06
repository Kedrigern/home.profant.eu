# Linux

`ls` → `eza`, `mv`, `cp`, `mkdir`, `tree`, `grep`

`cat` → `bat` , `tldr` 

`find`, `fd`, `column`

`htop`, `kill`

`vim`, `tmux`, `screen`

`ssh` (`~/.ssh/known_hosts` ), `rsync`, `rsync -av --exclude-from=.gitignore * <user>@<ip>:<dir>` 

`apg`

## Systemd

`systemctl <cmd> <service>` , kde cmd: `status`, `start`, `stop`, `restart`, `reload` ; service: `docker`, `nginx`, …

`journalctl`

`timedatectl set-local-rtc 1 --adjust-system-clock` oprava hodin

## Terminal

### Screen

```console
$ screen -s <name> # vytvoření pojmenované screeny
# nyní jsme v pojmenované screeně, můžeme udělat například:
$ while true; do date; sleep 0.9; done
[ctrl+a+d] # detach, odchod z aktuální screeny bez ukončení
$ screen -ls
$ screen -r <name> 
```

### Tmux

```console
$ tmux new -s <name>
$ tmux ls
[ctrl+b + d] # detach
$ tmux a -t <name>
$ tmux kill-ses -t <name>
```

`ctrl + b` nám dovolí zadávat příkazy:

`d` detach

`%` / `[shift+5]` vertikální rozdělení (vedle sebe)

`"` / `[shift +!]` horizontální rozdělení (nad sebou)

`arrow` přepínání mezi panely

`q` zobrazí čísla panelů

`q+n` přepnutí na panel číslo <n>

`ctrl + arrow` velikost panelů

`alt + arrow` velikost panelů (větší skok)

`w` přepínání mezi vším

`x` kill panel, window

### Zellij

## Kryptografie

```console
$ apg         # random password

# random
$ openssl rand -hex 32
$ openssl rand -base64 32

# hash
$ openssl dgst -md5 myfile.txt
$ openssl dgst -sha256 myfile.txt

# šifrování souboru
$ openssl enc -aes-256-cbc -salt -in myfile.txt -out myfile.enc
```

```console
ssh -p <port> <user>@<server>
ssh-keygen -t rsa -b 4096  # alg. RSA
ssh-keygen -t ecdsa -b 256 # algo. ECDSA (vyšší bezpečnost při menší velikosti klíče)
ssh-copy-id -i ~/.ssh/id_rsa.pub <user>@<server> # nakopírování klíče na server
```

```console
rsync [OPTIONS] source destination
rsync -avz --exclude='*.tmp' --exclude='.git' ~/project user@server:/var/backup
```

- `a`: Archivní mód (zachovává téměř všechna metadata).
- `v`: Podrobný výstup.
- `z`: Komprese dat během přenosu.
- `--ignore-existing`
- `--delete` odstranění souborů na cílovém serveru, které již nejsou na zdrojovém.

## Files a privileges

`useradd` (high level), `adduser`, `su <user>` , `sudo`, `who`, `chmod`, `chown`

| **Binary** | **Octal** | **Permissions** | **Representation** |
| ---------- | --------- | --------------- | ------------------ |
| 000        | 0 (0+0+0) | No permission   | \- - -             |
| 001        | 1 (0+0+1) | Execute         | \- - x             |
| 010        | 2 (0+2+0) | Write           | \- w -             |
| 011        | 3 (0+2+1) | Write + Ececute | \- w x             |
| 100        | 4 (4+0+0) | Read            | r - -              |
| 101        | 5 (4+0+1) | Read + Execute  | r - x              |
| 110        | 6 (4+2+0) | Read + Write    | rw -               |
| 111        | 7 (4+2+1) | R + W + E       | rwx                |

### Find

```console
# Vyhledávání podle názvu
find . -name "*.txt"          # Hledá .txt soubory
find . -iname "*config*"      # Hledá case insensitive

# Vyhledávání podle typu
find . -type d                # Najde adresáře
find . -type f                # Najde soubory

# Vyhledávání podle času
find . -mtime -7              # Upravené v posledních 7 dnech

# Vyhledávání podle velikosti
find . -size +100M            # Větší než 100MB

# Exekuce příkazů
find . -name "*.log" -exec rm {} \\;  # Smaže všechny .log soubory
```

### fd - alternativa find

- `fd` má lepší výchozí nastavení (ignoruje .git, respektuje .gitignore)
- `fd` je rychlejší a má přehlednější syntaxi
- `fd` má automaticky barevný výstup
- `fd` lépe podporuje Unicode a PCRE regex
- `find` je dostupný všude bez instalace
- `find` má více pokročilých možností

```console
# Jednoduché vyhledávání
fd dokument               # Najde soubory obsahující "dokument"
fd -e pdf                 # Najde všechny PDF soubory
fd -e jpg -e png          # Najde JPG a PNG soubory

# Vyhledávání podle typu
fd -t f                   # Pouze soubory
fd -t d                   # Pouze adresáře
fd -t l                   # Pouze symlinky
fd -t x                   # Pouze spustitelné soubory

# Omezení hloubky
fd -d 2 pattern           # Hledá do hloubky 2 adresářů

# Ignorování a výjimky
fd -I pattern             # Ignoruje soubory z .gitignore
fd -E "*.bak"             # Vyloučí .bak soubory

# Spouštění příkazů
fd -e txt -x wc -l        # Počítá řádky v .txt souborech
fd -e jpg -x convert {} {.}.png  # Konvertuje JPG na PNG

# Časové vyhledávání
fd -t f --changed-within 24h  # Soubory změněné za 24h

# Prázdné soubory/adresáře
fd -t f -s                # Prázdné soubory
fd -t d -s                # Prázdné adresáře
```