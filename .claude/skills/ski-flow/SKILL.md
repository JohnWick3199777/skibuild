---
name: ski-flow
description: Translates research output into a concrete CLI command surface and stage-by-stage execution plan. Use ski-flow immediately after ski-research completes, or whenever the user asks to "plan the build", "define the flow", "figure out the commands", "map out the stages", or wants to know what to implement before writing any code. Always invoke ski-flow before ski-env, ski-dev, or any implementation work begins — if there's a research report in .ski/ski-research/, this skill should run next.
---

# ski-flow

ski-flow reads research output and turns it into two things: a concrete **command surface** (what commands exist, what arguments they accept) and a **stage-by-stage plan** (what each pipeline module needs to do). Downstream modules — ski-env, ski-dev, ski-test, ski-docs — consume this plan so they never have to re-read the research.

**This is a human-in-the-loop process.** Never write the final plan and move on — present each layer to the user, wait for approval, incorporate their edits, then proceed. The user is the decision-maker; you are the translator from research to plan.

---

## Step 1: Read the research and map the architecture

Read `.ski/ski-research/research.md`. If it doesn't exist, stop and ask the user to run ski-research first.

Extract and summarize the architecture before doing anything else — the command surface only makes sense once you understand what you're wrapping.

**Components** — what are the distinct pieces of the system?
- Is there a **server** (long-running daemon, background process) separate from a **client** (CLI, SDK caller)?
- Or is it a single-process tool with no server component?
- Are there other components — agent, worker, proxy, broker?

**Communication** — how do components talk to each other?
- How does the client locate the server? (Unix socket, HTTP port, env var, config file)
- What protocol? (REST, gRPC, WebSocket, IPC, CLI subprocess)
- Is there auth between client and server?

**Ownership** — what does each component own?
- What state, connections, or file handles does the server hold?
- Is the client stateless between calls?

**Interaction surface** — what can the client actually ask the server to do?
- API endpoints, SDK methods, CLI flags, events

**Gaps** — unresolved questions from the research that need design decisions

Use this understanding to drive the command groups in Step 2 — the `service` group maps to the server component, the `interact` group maps to the client→server interaction surface, and so on.

---

## Step 2: Propose command groups — wait for approval

Before defining individual commands, group them by area of concern. Start from the standard groups below, then add or remove based on what the research actually supports.

### Standard command groups

**service** — lifecycle of the background process. Include this group whenever the target runs as a long-lived daemon or server.
```
<cli> start     Launch the service in the background
<cli> stop      Gracefully shut it down
<cli> restart   Stop + start in one step
<cli> status    Show whether it's running, its PID, port, and uptime
<cli> attach    Stream live logs from the running service (tail -f style)
```
Service state (PID, port, socket path, start time) is written to `~/<cli>/<cli>_service.json` on start and removed on stop. `status` and `attach` read from this file — they never probe the process directly.

**auth** — credentials and session management. Include when the target requires authentication.
```
<cli> auth login    Authenticate and persist a token
<cli> auth logout   Clear stored credentials
<cli> auth status   Show current auth state (logged in / token expiry)
```
Credentials are stored in `~/<cli>/credentials.json`, never in env vars or plain text config.

**config** — user-facing settings. Include when the target has meaningful configuration options (base URL, default flags, output preferences).
```
<cli> config set <key> <value>   Write a setting
<cli> config get <key>           Read a setting
<cli> config list                Show all current settings
```
Config is stored in `~/<cli>/config.json`.

**interact** — commands that talk to the running service. Shape these entirely from the research — what the target's API or interaction surface actually supports.
```
<cli> get <resource>     Fetch data from the running service
<cli> push <resource>    Send data or trigger an action
<cli> watch <resource>   Stream live updates (if the target supports it)
```
These commands require the service to be running. If it isn't, surface a clear error: `"service is not running — start it with '<cli> start'"`.

**export / output** — produce files or artifacts. Include when the target generates output the user keeps.
```
<cli> export <input>    Convert or save output to a file
  --format  STR   Output format (default depends on target)
  --out     PATH  Output destination
```

---

Present the proposed groups to the user with a one-line rationale for each one included or excluded:

```
Based on the research, here are the proposed command groups:

  service   ✓  target runs as a background server
  auth      ✓  API requires a token (from research: "requires API key")
  config    ✓  base URL and output format are user-configurable
  interact  ✓  REST API surface documented in research
  export    ✓  target produces SVG/PNG artifacts

  Hidden state files:
    ~/.excalidraw/excalidraw_service.json   (PID, port, start time)
    ~/.excalidraw/credentials.json          (API token)
    ~/.excalidraw/config.json               (user settings)

Does this grouping look right? Any groups to add, remove, or rename?
```

**Wait for the user to respond.** Don't proceed to individual commands until they've confirmed or adjusted the groups.

---

## Step 3: Propose commands within each group — wait for approval

For each approved group, propose the individual commands. Present all groups together in one message so the user can review the full surface at once.

Format each command with its arguments and flags. Mark anything not directly from the research as `[inferred]`:

```
service
  <cli> start
    --port    INT    Port to bind (default: from research or 3000)
    --host    STR    Host interface (default: localhost)   [inferred]
    --detach  BOOL   Run in background, don't stream logs (default: true)   [inferred]

  <cli> stop
    --force   BOOL   Kill immediately without graceful shutdown (default: false)   [inferred]

  <cli> status      (no flags — reads ~/.cli/cli_service.json)

  <cli> attach      (no flags — streams logs from running service)

auth
  <cli> auth login
    --token   STR    Pass token directly instead of interactive prompt   [inferred]

  <cli> auth logout  (no flags)
  <cli> auth status  (no flags)

config
  <cli> config set <key> <value>
  <cli> config get <key>
  <cli> config list

interact
  <cli> get <resource>
    --format  STR    Response format: json|table (default: table)   [inferred]

export
  <cli> export <input-file>
    --format  STR    Output format: svg|png|json (default: svg)
    --out     PATH   Output path (default: <input>.svg)
```

Then ask:

```
Does this look right? For each command you want to change, tell me:
- Add or remove a flag
- Rename a command
- Change a default
- Drop a command entirely
```

**Wait for approval.** Incorporate any changes and re-show the affected commands before proceeding. The user should feel like they're signing off on the CLI contract.

---

## Step 4: Confirm the stage plan

Once the command surface is approved, briefly outline what each pipeline stage will do. Keep it short — this is a sanity check, not a deep spec:

```
env   → scaffold project, install deps, create folder structure
dev   → implement the approved commands, wire up the CLI entry point
test  → smoke-test each command; validate outputs match PoC results
docs  → generate SKILL.md with usage examples for the approved commands
```

Ask: "Anything to adjust before I write the plan?"

Wait for a response. If they're happy, proceed.

---

## Step 5: Write the plan

Only after the command surface and stage outline are approved, save `.ski/ski-flow/plan.md`:

```markdown
# Flow Plan: <Skill Name>

## Command surface

<approved commands and flags — [inferred] labels preserved>

## State files

| File | Purpose |
|------|---------|
| `~/<cli>/<cli>_service.json` | PID, port, socket path, start time — written by `start`, removed by `stop` |
| `~/<cli>/credentials.json`  | Auth token — written by `auth login`, cleared by `auth logout` |
| `~/<cli>/config.json`       | User settings — managed by `config set/get/list` |

<omit rows for groups not included in this skill>

## Stage plan

### env
- Scaffold: <files/dirs to create>
- Install: <exact install command, e.g., npm install excalidraw@0.17>
- State dir: ensure `~/<cli>/` is created on first run, not at install time

### dev
- Entry point: <file path>
- Commands to implement: <list, matching approved surface>
- Integration: <how the CLI connects to the target — API base URL, SDK import, subprocess call>
- Service state pattern: start writes `~/<cli>/<cli>_service.json`; status/attach read it; stop removes it

### test
- Smoke test: <exact command to run>
- Validate: <what to check — exit code, HTTP response, state file contents>

### docs
- Key usage examples: <the 2-3 commands a user would actually run>

## Assumptions
- [assumed]: <flag or behavior assumed without research confirmation, and why>

## Open decisions
- [ ] <anything still unresolved — surface to user if blocking>
```

---

## Step 6: Tell the user what's next

After writing the plan, confirm:

```
Plan written to .ski/ski-flow/plan.md.

Next step: ski-env will scaffold the project using this plan.
Run ski-env when you're ready.
```

---

## Quality checks

- Command groups were shown and approved before individual commands were designed
- Individual commands were shown and approved before the plan was written
- Every flag traces to the research or is labeled `[inferred]`
- The plan is self-contained: ski-env can scaffold from `plan.md` without reading `research.md`
- Nothing was assumed silently — all assumptions are listed in the plan
