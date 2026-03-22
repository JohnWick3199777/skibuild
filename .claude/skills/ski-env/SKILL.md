---
name: ski-env
description: Scaffolds a new CLI project from a ski-flow plan. Sets up the UV Python environment, initializes a git repo, generates the project structure, and produces an install.sh for one-command global installation via uv tool install. Use this after ski-flow has produced an approved plan, or whenever the user asks to "scaffold", "set up the project", "initialize the env", or "create the folder structure". Always run ski-env before ski-dev.
---

# ski-env

ski-env takes the approved flow plan and turns it into a real project on disk: a UV-managed Python package, a git repo, the folder structure, and an `install.sh` so anyone can install the CLI globally in one command.

Every CLI built with skibuild is:
- A **UV project** (`pyproject.toml` + `uv` for all dependency and env management — no virtualenv, no pip, no conda)
- A **globally installed tool** via `uv tool install` — the CLI lives on `$PATH` after install, no activation needed
- A **git repo** from day one, versioned inside the project itself
- **Easy to install** — `install.sh` is the single entry point for anyone picking up the project

---

## Step 1: Read the plan

Read `.ski/ski-flow/plan.md`. Extract:
- CLI name (used for the package name, command name, and state directory `~/<cli>/`)
- Dependencies from the stage plan's `env` section
- Commands from the approved command surface

If the plan doesn't exist, stop and ask the user to run ski-flow first.

---

## Step 2: Scaffold the project

Create the project at the path the user specifies (or ask if not clear). Structure:

```
<cli-name>/
├── pyproject.toml          ← UV project manifest + CLI entry point
├── README.md               ← one-paragraph description + install instructions
├── install.sh              ← one-command global installer
├── .python-version         ← pins Python version for UV
├── .gitignore
└── src/
    └── <cli_name>/
        ├── __init__.py
        ├── main.py         ← CLI entry point (Typer or Click app)
        ├── service.py      ← service lifecycle (start/stop/status/attach) if service group included
        ├── auth.py         ← auth commands if auth group included
        ├── config.py       ← config commands if config group included
        └── interact.py     ← interact/export commands
```

Only create the files that correspond to the approved command groups. Don't scaffold `auth.py` if auth was excluded from the plan.

---

## Step 3: Write pyproject.toml

Use UV's project format. The `[project.scripts]` entry wires the CLI name to the entry point so `uv tool install` puts it on `$PATH`:

```toml
[project]
name = "<cli-name>"
version = "0.1.0"
description = "<one-line description from plan>"
requires-python = ">=3.11"
dependencies = [
    "click>=8.1",
    # add deps from plan here
]

[project.scripts]
<cli-name> = "<cli_name>.main:cli"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

Pin the Python version in `.python-version`:
```
3.11
```

---

## Step 4: Write install.sh

`install.sh` is the primary install path. It should:
1. Check that `uv` is installed; if not, print a one-liner to install it and exit
2. Run `uv tool install .` from the project root
3. Print a success message with the command name and a usage hint

```bash
#!/usr/bin/env bash
set -e

if ! command -v uv &>/dev/null; then
  echo "uv is not installed. Install it first:"
  echo "  curl -LsSf https://astral.sh/uv/install.sh | sh"
  exit 1
fi

echo "Installing <cli-name>..."
uv tool install .
echo ""
echo "<cli-name> installed. Try: <cli-name> --help"
```

Make it executable: `chmod +x install.sh`.

---

## Step 5: Initialize git

Inside the project directory:

```bash
git init
git add .
git commit -m "init: scaffold <cli-name> with ski-env"
```

The project is versioned from the first commit. Every subsequent ski-dev implementation and ski-test fix should be committed too — the git history is the project's changelog.

---

## Step 6: Wire the entry point

In `src/<cli_name>/main.py`, create a Click group with placeholder subgroups matching the approved command surface:

```python
import click
from <cli_name> import service, auth, config, interact  # only what was scaffolded

@click.group()
def cli() -> None:
    """<description>"""

# add subgroups for each approved command group
cli.add_command(service.service)
cli.add_command(auth.auth)
cli.add_command(config.config)
cli.add_command(interact.interact)

if __name__ == "__main__":
    cli()
```

Each submodule exports a `@click.group()` with stubbed `@click.command()` functions that just print `"not yet implemented"`. ski-dev fills in the logic.

**Click patterns to follow across all submodules:**
- Use `@click.group()` for command groups (service, auth, config)
- Use `@click.command()` for individual commands (start, stop, login)
- Use `@click.option()` for flags, `@click.argument()` for positional args
- Annotate all function signatures with types — `uv ty` will check them

---

## Step 7: Verify the scaffold

Run both checks before handing off to ski-dev:

```bash
# entry point resolves and help renders
uv run <cli-name> --help

# type check passes on the stub code
uv run ty check src/
```

Both must pass. If `uv ty` reports errors on the stub code, fix the type annotations before proceeding — ski-dev inherits these as the baseline.

---

## Step 8: Write the env report

Save `.ski/ski-env/report.md`:

```markdown
# Env Report: <CLI Name>

## Project path
<absolute path to the scaffolded project>

## Commands scaffolded
<list of command groups and commands, matching the approved surface>

## Install
cd <project-path> && ./install.sh

## State files (created at runtime, not by scaffold)
~/<cli>/<cli>_service.json
~/<cli>/credentials.json
~/<cli>/config.json

## Git
Initialized. Initial commit: "init: scaffold <cli-name> with ski-env"

## Next step
Run ski-dev to implement the command logic.
```

---

## Quality checks

- `uv run <cli-name> --help` works before handing off to ski-dev
- `install.sh` is present, executable, and works from a clean directory
- git is initialized with at least one commit
- Only files for approved command groups were created — no dead stubs
- State files (`~/<cli>/`) are NOT created by the scaffold — they're created at runtime by the CLI itself
- pyproject.toml uses `[project.scripts]` so `uv tool install` puts the CLI on `$PATH`
