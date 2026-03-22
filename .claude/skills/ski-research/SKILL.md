---
name: ski-research
description: Research layer for skibuild. Use this skill whenever the user wants to investigate a library, program, tool, or API in preparation for building something — a CLI, a skill, a wrapper, an integration. Triggers on prompts like "research X so I can build Y", "I want to create a tool using X", "how does X work", or any exploration that precedes implementation. Always use this skill when the user names a target technology and a build intent, even if they don't use the word "research".
---

# ski-research

The goal is to produce a research report thorough enough that implementation can start without stopping to look things up. Good research means: you know how to run the target, you know how to talk to it, and you've validated the approach with a PoC.

## Research process

### Step 1: Parse the intent

Before searching anything, extract from the user's prompt:

- **Target** — what library/program/API is being researched (e.g., `excalidraw`)
- **Goal** — what the user wants to build with it (e.g., "CLI to run a local instance and interact with it")
- **Constraints** — language, platform, existing stack (or note "none stated")

If these are ambiguous, ask before searching.

### Step 2: Web research

Search systematically, in this order:

1. **Find the official source** — GitHub repo, npm/PyPI/crates page, official site
   - Queries: `"<target> github"`, `"<target> npm"`, `"<target> documentation"`
   - Fetch and read the README and any "getting started" or "self-hosting" pages

2. **Understand the architecture** — what are the distinct components?
   - Is there a **server** (long-running process, daemon, background service) and a **client** (CLI, SDK, HTTP caller) that are separate things?
   - Or is it a single-process library / standalone CLI with no server component?
   - If there's a server: what does it own? (state, connections, file handles, ports)
   - If there's a client: how does it find and talk to the server? (Unix socket, HTTP, gRPC, named pipe, env var pointing to a socket path)
   - Are there other components? (agent, worker, proxy, broker, registry, sidecar)
   - Look for architecture diagrams, "how it works" sections, and deployment docs

3. **Understand the runtime model** — how does each component run?
   - How is the server started and stopped? Does it daemonize itself or require a process manager?
   - What ports or socket paths does it use by default? Are they configurable?
   - Does the client need to be installed separately from the server, or is it the same binary?
   - Look for "local development", "self-host", "run locally" docs

4. **Find the interaction surface** — how does the client talk to the server?
   - REST API? WebSocket? SDK? CLI flags? Unix socket? gRPC? IPC?
   - What does a minimal request/response look like?
   - Is auth required between client and server (token, mTLS, socket permissions)?
   - Look for API docs, OpenAPI specs, example scripts, SDK references

5. **Find community examples** — see how others have done similar things
   - Queries: `"<target> cli wrapper"`, `"<target> local instance"`, `"<target> programmatic"`

Don't just collect URLs — read the pages and extract the key facts. For long docs, focus on the sections most relevant to the goal.

### Step 3: Write and run PoC scripts

After web research, write small scripts to validate the approach. These are probes, not production code. Run them and record what happens.

Typical PoCs for a "run locally + interact" goal:

- **Launch PoC** — install and start the target, confirm it's running
  ```bash
  # example
  npx serve-excalidraw  # or whatever the correct package/command is
  ```
- **Interaction PoC** — send a minimal request or call to the running instance
  ```bash
  curl http://localhost:3000/api/...
  ```
- **Teardown check** — confirm the process can be cleanly stopped

Save PoC scripts to `.ski/ski-research/poc/` so they're available for the implementation phase without cluttering the user's working directory. Record stdout/stderr and whether the PoC succeeded.

### Step 4: Fill gaps

After research and PoC, identify what's still unknown or unvalidated. Be explicit — don't leave gaps hidden in prose.

## Output format

All output goes to `.ski/ski-research/` — never into the user's working directory. Create the directory if it doesn't exist. Write the research report to `.ski/ski-research/research.md` and summarize key findings in the conversation.

```markdown
# Research Report: <Target>

## Intent
- Target: <library/program>
- Goal: <what we're building>
- Constraints: <constraints, or "none stated">

## Architecture

### Components
| Component | Role | Process type |
|-----------|------|-------------|
| server    | <what it owns — state, connections, file handles> | long-running daemon / embedded / n/a |
| client    | <how it talks to the server> | CLI / SDK / HTTP caller |
| <other>   | <agent, worker, proxy, etc. if present> | ... |

If there is no server/client split (e.g. a pure library or single-process CLI), say so explicitly: "Single-process — no separate server component."

### How components communicate
<How does the client find the server? Unix socket path? HTTP port? gRPC? Named pipe? Env var pointing to a socket?>
<What does auth between client and server look like, if anything?>
<Example of a minimal client→server call, in pseudocode or curl>

### What the server owns
<State, open connections, file handles, ports — what is lost if the server is killed hard?>

### What the client does
<Is the client stateless? Does it hold any local state between calls? Where?>

## How to run locally
<concrete steps — install command, start command, default port/interface>
<If server and client are separate: show both>

## Interaction surface
<how to programmatically interact — API endpoints, SDK calls, CLI flags, WebSocket events, etc.>
<Focus on the client→server boundary specifically>

## Dependencies & requirements
<runtime version, OS, auth, env vars, etc.>

## Sources
| Source | URL | What it covers |
|--------|-----|----------------|
| GitHub | ... | Source + README |
| Docs   | ... | API reference  |

## PoC results
| Script | Outcome | Notes |
|--------|---------|-------|
| launch.sh | ✅ success | starts on port 3000 |
| interact.sh | ⚠️ partial | endpoint exists but auth required |

## Gaps & open questions
- [ ] <unresolved question 1>
- [ ] <unresolved question 2>
```

## Quality checks

- Every major claim traces to a source URL in the Sources table
- At least one PoC was run and its result is recorded
- Gaps are listed explicitly — if something couldn't be confirmed, say so
- The Architecture section explicitly answers: is there a server/client split? If yes, how do they communicate?
- The report is self-contained: a fresh reader can start implementing from it alone, including knowing exactly which components to build and how they connect
