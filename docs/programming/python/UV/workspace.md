# uv workspace

Every package in workspace:

- has same venv (`uv python find`)
- has same dependency tree (`uv tree`)
- cannot have conflict dependency

Advantages:

- you have always only one version of dependency
- so you can easily update dependencies

Typical structure:

```
packages/
├── package1
│   ├── pyproject.toml
│   └── src
├── package2
│   ├── pyproject.toml
│   └── src
├── pyproject.toml
└── uv.lock
```

- only one `uv.lock`
- structure can be deeeper like: `packages/category/package1/pyproject.toml`
- from `package1` etc you need to add root project package as dependency (to have common dependencies)

## Commands

Recomended commands:

Create root package:  
`uv init --vcs none --name <name> --bare --no-pin-python .`

Create member package:  
`uv init --vcs none --name <name> --package --no-pin-python --build-backend uv .`

Show outdated packages:  
`uv tree --outdated`

Update packages:  
`uv sync -U`

## Pyproject

Root package has defined all members:

```toml
[tool.uv.workspace]
members = [
    "package1",
]
```

Member package:


```toml
dependencies = [
    "workspace",        # root package
    ...
]

[tool.uv.sources]
workspace = { workspace = true }
```

## Example

```python
#!/usr/bin/env bash

workdir=simpleworkspace

if [ -d "$workdir" ]; then
        echo "$workdir dir already exists"
        exit 1;
fi

mkdir -p "$workdir"/package1 && cd "$workdir"

echo "== Prepare project structure =="

uv init -q --vcs none --name workspace --bare --no-pin-python .
uv add -q --dev "ruff==0.10.0"  # old version of ruff

cd package1
uv init -q --vcs none --name package1 --package --no-pin-python --build-backend uv .
cd ..

uv add -q --package package1 workspace "cowsay==6.0"

uv sync -q

uv build -q --all-packages

echo "== Result =="
echo "Dependencies:"

uv tree --outdated

echo "== Sumary =="

echo -n "UV version: "
uv -V
echo -n "Venv: "
uv python find
```
