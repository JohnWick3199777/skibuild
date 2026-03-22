---
name: ski-test
description: Runs the verification gate on a CLI built by ski-dev — type checks with uv ty, entry-point validation, and smoke tests against the approved command surface. Use after ski-dev has implemented the commands, or whenever the user asks to "run tests", "check types", "verify the CLI", or "validate the build". A clean ski-test run is required before ski-docs.
---

# ski-test

ski-test runs three layers of checks in order. All three must pass before the CLI moves to ski-docs. Failures block export and must be fixed by ski-dev.

---

## Layer 1: Type check with uv ty

```bash
uv run ty check src/
```

`uv ty` is the type checker — always use it, never mypy, pyright, or anything else.

Fix all errors before proceeding to layer 2. Common things to check:
- All Click command functions have typed signatures (`str`, `int`, `bool`, `Path`)
- Return types are annotated (commands return `None`)
- Any data structures passed between modules are typed (dataclasses, TypedDicts, or Pydantic models)

If `ty` is not yet installed, add it: `uv add --dev ty`

---

## Layer 2: Entry-point validation

Confirm the CLI installs and the help tree renders correctly:

```bash
# check the package is importable and the entry point resolves
uv run <cli-name> --help

# check each top-level command group
uv run <cli-name> service --help
uv run <cli-name> auth --help       # if included
uv run <cli-name> config --help     # if included
```

Every approved command group must appear in `--help` output. Any missing or broken group is a ski-dev issue, not a test issue — send it back.

---

## Layer 3: Smoke tests

Run a minimal end-to-end test for each command group. The goal is to confirm the command runs and produces the expected behavior — not to test every edge case.

### service group
```bash
# start the service, confirm state file is written
uv run <cli-name> start
test -f ~/.<cli-name>/<cli-name>_service.json && echo "state file ✓"

# status shows running
uv run <cli-name> status

# stop cleans up the state file
uv run <cli-name> stop
test ! -f ~/.<cli-name>/<cli-name>_service.json && echo "state file removed ✓"
```

### auth group (if included)
```bash
# login writes credentials file
uv run <cli-name> auth login --token test-token
test -f ~/.<cli-name>/credentials.json && echo "credentials file ✓"

# status reflects logged-in state
uv run <cli-name> auth status

# logout clears credentials
uv run <cli-name> auth logout
test ! -f ~/.<cli-name>/credentials.json && echo "credentials cleared ✓"
```

### config group (if included)
```bash
uv run <cli-name> config set base_url http://localhost:3000
uv run <cli-name> config get base_url   # should print http://localhost:3000
uv run <cli-name> config list
```

### interact / export groups
Run the smoke test from the flow plan's test section — the exact command and validation defined there. If the plan doesn't have one, ask ski-dev to add it.

---

## Write the test report

Save `.ski/ski-test/report.md`:

```markdown
# Test Report: <CLI Name>

## uv ty
- Result: ✅ pass / ❌ fail
- Errors: <list if any>

## Entry-point validation
- Result: ✅ pass / ❌ fail
- Commands checked: <list>
- Missing or broken: <list if any>

## Smoke tests
| Command | Result | Notes |
|---------|--------|-------|
| start   | ✅     | state file written at expected path |
| stop    | ✅     | state file removed |
| auth login | ✅  | credentials.json written |
| ...     | ...    | ... |

## Verdict
✅ All checks passed — ready for ski-docs.
❌ Blocked — see failures above. Return to ski-dev.
```

---

## Quality checks

- `uv run ty check src/` is always the first check run — never skipped
- Smoke tests validate actual file system state (state files, credentials) not just exit codes
- A failed layer blocks the next — don't run smoke tests if type check fails
- Failures are specific: which command, which assertion, what was expected vs. what happened
