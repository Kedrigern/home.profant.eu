---
title: Git
icon: fontawesome/brands/git-alt
tags:
  - devops
  - cvs
---

```console
$ git init
$ git add <files>
$ git commit -m <message>

git stash push -m "popis"
git stash list
git stash pop

git gui

git branch --list       # výpis
git branch <brname>     # nová větev
git checkout <brnamne>  # změna větve

git branch -d <název_lokální_větve>  # Smaže, pokud je mergovaná
git branch -D <název_lokální_větve>  # Smaže bez ohledu na stav (používejte opatrně)

# Set a new remote
git remote add origin https://github.com/OWNER/REPOSITORY.git
git remote -v  # Verify new remote

git log --graph --decorate --oneline
```

```console
$ uv init
$ git init
$ git add hello.py .gitignore pyproject.toml README.md
$ git commit -m "Initial commit"
$ git branch dev    # Vytvoření dev větve
$ git checkout dev  # Práce v dev větvi
$ git checkout main # Práce v main větvi (hotfix)
$ git merge
Auto-merging hello.py
Merge made by the 'ort' strategy.
 hello.py | 2 ++
 1 file changed, 2 insertions(+)
$ git branch --list
  dev
* main
$ git log --graph --decorate --oneline
*   7acddf8 (HEAD -> main) Merge branch 'dev'
|\  
| * 73c74de (dev) Add dev function
* | c3cb8cc Fix the msg on prod
|/  
* f430b2e Initial commit
```

## Multiple keys

```console
$> cat ~/.ssh/config
# Work account
Host github.com-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/my_company_key_ed25519

# Personal account
Host github.com-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/my_personal_key_ed25519
```

Then use the corresponding host when cloning repositories:

```console
$ git clone git://github.com-work/company/your-work-repo.git
```

## Github
