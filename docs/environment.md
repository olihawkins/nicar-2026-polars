# Environment

Thsese are the to reproduce the virtual environment, both from the lockfile and from scratch if necessary.

## Create the environment from the lockfile

Use `uv sync` to recreate the environment from the lockfile.

```zsh
uv sync
```

## Create the environment from scratch

Initialise the project.

```zsh
uv init --python ">=3.13"
```

Create a new virtual environment: the seed argument installs `pip` into the venv.

```zsh
uv venv --seed
```

Add the required python packages

```zsh
uv add ipython jupyterlab polars
```

## Activate and deactivate the environment

Activate the enironment.

```zsh
source .venv/bin/activate
```

Deactivate the enironment.

```zsh
deactivate
```
