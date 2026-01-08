---
title: uv
icon: simple/uv
---

An extremely fast Python package and project manager, written in Rust. It replaces pip, pipx, PyEnv and Poetry.

TODO: Performance comparation

## uvx

Via `uvx` you can run packages in any Python version without manual installation of the package.
As you can see, the Python version can even be unreleased.

```console
$ uvx <package> <package_args>
$ uvx --python 3.14 cowsay -t "Hello world from uvx"
$ uvx poetry@1.8 -V      # run old version of poetry
```

Or if you want to test a dependency:

```console
$ uv run --with httpx==0.26.0 python -c "import httpx; print(httpx.__version__)"
0.26.0
$ uv run --with httpx==0.25.0 python -c "import httpx; print(httpx.__version__)"
0.25.0
```

## Basic usage

```console
$ uv python install 3.14

$ uv init                 # create pyproject.toml, initialize venv and git
$ uv add requests
$ uv add --dev ruff pytest
$ uv run <script>         # run script or module
$ uv sync                 # install dependencies from lockfile
```

## Intermediate usage

### Package management

```console
$ uv add <package>
$ uv remove <package>
$ uv add --dev <package>
$ uv add -r requirements.txt                         # add dependencies from requirements.txt

$ uv pip list --outdated --format json
$ uv tree                                            # dependency tree
$ uv tree --depth 0 --outdated --package <package>   # info about package
# handy are also args: --no-dev and --only-dev
```

### Upgrade package

```console
$ uv init --bare
$ uv add "colorama==0.4.5"  # latest 0.4.6
$ uv sync -U                # nothing to do, because explicit version of dependency
$ uv add "colorama<1"
$ uv sync -U                # install 0.4.6
```

### uv.lock

#### uv sync

- Behaves smartly for developers.   
- It does not fail if `uv.lock` is missing; instead, it calls `uv lock` itself to create it.  
- If uv.lock is outdated, it updates it according to pyproject.toml.  
- Commands like `uv run`, `uv add`, and `uv remove` also ensure the environment is synchronized.  

#### uv sync --locked

- This is the strict variant.   
- It requires the uv.lock file to exist.  
- It requires pyproject.toml and uv.lock to be in sync.   
- Equivalent to `poetry lock --check` and `poetry install`

#### uv sync --frozen

- ignore that `pyproject.toml` has changed, use only frozen `uv.lock`

- > Instead of checking if the lockfile is up-to-date, **uses the versions in the lockfile as the source of truth**. If the lockfile is missing, uv will exit with an error. If the pyproject.toml includes changes to dependencies that have not been included in the lockfile yet, they will not be present in the environment.

```console
$ uv init --bare
$ uv add "colorama"
$ rm uv.lock
$ uv sync --frozen          # error
$ uv sync --locked          # error
$ uv sync                   # recreate uv.lock
```


### pyproject.toml

```toml
[project]
name = "package-name"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
authors = [{ name = "", email = "" }]
classifiers = ["Private :: Do Not Upload"] # Prevents accidental uploads to PyPI.
requires-python = ">=3.12"
dependencies = [
    "python-dotenv>=1.1.1"
]

[project.scripts]
# entrypoint, be careful - hyphens are not allowed as python module names, so you need to replace them with _
package-name = "package_name.app:app"        

[[tool.uv.index]]
name = "pytorch-cu124"
url = "https://download.pytorch.org/whl/cu124"

# In the near future it will be replaced by uv native build
# uv init --build-backend uv
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[dependency-groups]
dev = [
    "pytest>=8.4.1",
    "ruff>=0.12.1",
]
```

### Python and venv

```console
$ uv python list          # list available Python versions
$ uv python install 3.9   # install Python version
$ uv python find          # currently used Python and venv
```

### Credentials

```console
# Credentials for read index:
$ export UV_INDEX_<index_name>_USERNAME=public
$ export UV_INDEX_<index_name>_PASSWORD=koala

# Credentials for publish index:
$ export UV_PUBLISH_USERNAME=""
$ export UV_PUBLISH_PASSWORD=""

# or .env
$ source ~/.env

# or netrc
# is loaded automatically after obtaining 401 status code
$ cat ~/.netrc
machine company.jfrog.io # index domain without protocol
login <login>
password <password>
```

### Deployment

```console
$ uv sync --no-dev --frozen    # sync to lockfile on production
```

### Publishing packages

You need to have a built package: 

```console
$ uv version              # print current project version
$ uv version "1.0.0"      # set new version to project
$ uv build                # build package to the `dist` dir
$ uv publish --index <index_name> -u "user" -p "passwd"
```

### Workspace

[uv workspace](workspace.md)

### requires-python vs .python-version

requires-python: range of compatible Python versions  
.python-version: requested exact Python version

- for example, your distribution version, not the newest
- it may not be versioned in git

So you always need `requires-python` in `pyproject.toml`. If you need a special version of Python to run (for example, for performance reasons), add a `.python-version` file beside `pyproject.toml`.

## Full examples

### Testing compatibility

You want test your app in unreleased python 3.14 with older version of lib? (current httpx 0.28.*) No problem:

```console
$ uvx --python 3.14 --with httpx==0.25.0 cowsay -t "Hello World
```

### Quick acces to funcionality

Run repl in specific python version with polars:

```
uv run --with polars --python 3.13  python
```

### FastAPI

```console
$ uv init --app
$ uv add fastapi --extra standard
$ uv run fastapi dev
...
$ uv run uvicorn main:app
```

### Wagtail

```console
$ uvx wagtail start mysite
$ cd mysite
$ uv add wagtail
```

### Container

```console
$ mkdir my-app && cd my-app
$ uv init --vcs none --package --build-backend uv --no-pin-python --no-workspace .
```

Add the `.dockerignore`:
```txt
.git
.venv
tests
Dockerfile
```

Add the `Dockerfile`:

```Dockerfile
# Debian trixie will be released soon.
FROM python:3.12-slim-bookworm

ARG UV_INDEX_<index_name>_USERNAME
ARG UV_INDEX_<index_name>_PASSWORD

RUN apt-get update && apt-get install -y dumb-init && rm -rf /var/lib/apt/lists/*

COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/
# Exact version:
# COPY --from=ghcr.io/astral-sh/uv:0.7.19 /uv /uvx /bin/

ADD . /app

WORKDIR /app

ENTRYPOINT ["/usr/bin/dumb-init", "--"]

# if the script is defined in pyproject.toml:
CMD ["uv", "run", "my-app"]
```

```console
$ uv run my-app
Hello from my-app!
$ podman build -t my-app .
WARN[0000] missing "UV_INDEX_<index_name>_USERNAME" build argument. Try adding "--build-arg UV_INDEX_<index_name>_USERNAME=<VALUE>" to the command line
WARN[0000] missing "UV_INDEX_<index_name>_PASSWORD" build argument. Try adding "--build-arg UV_INDEX_<index_name>_PASSWORD=<VALUE>" to the command line
...
Successfully tagged localhost/my-app:latest
<image_full_id>
$ podman run -it --rm localhost/my-app
Hello from my-app!
$ podman run -it --rm localhost/my-app bash
# ...
```

## Additional notes

uv also makes available many non-Python apps. For example, git-cliff is a Rust application but it is published on PyPI. So you can easily run:

```console
uvx git-cliff ...
```

(Of course this is not uv specific, but with uv fast it is interesting)

## Python and PATH

uv installs Python versions in `.local/share/uv/python/`. Unlike pyenv, it doesn’t automatically add the installed Python executables to your system’s `PATH`. This means you can’t just run python directly unless you’ve set up a symlink or manually adjust your PATH.
