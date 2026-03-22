# skibuild

skibuild is the core engine for building, refining, and exporting AI skills in the ski.ai ecosystem. It orchestrates a skill's full lifecycle — from initial research through implementation, testing, and documentation — via a modular, CLI-driven pipeline.

## Two top-level workflows

- **`ski init`** — builds a new skill from scratch: research → flow → env → dev → test → docs
- **`ski refine`** — iterates on an existing skill through the same pipeline stages

## Module pipeline

Each module is a discrete stage. Output from one feeds the next.

```
ski-research → ski-flow → ski-env → ski-dev → ski-test → ski-docs
```

| Module | Role |
|--------|------|
| `ski-research` | Web research, doc parsing, PoC scripts — produces a research report |
| `ski-flow` | Translates research into an execution plan and stage-by-stage flow |
| `ski-env` | Scaffolds the project: git, folder structure, boilerplate |
| `ski-dev` | Implements and iterates on the skill logic |
| `ski-test` | Type checks, entry-point validation, smoke tests |
| `ski-docs` | Generates SKILL.md, README.md, CLAUDE.md, and export metadata |

## Directory conventions

Skill definitions (source of truth) live in `.claude/skills/`:

```
skibuild/
└── .claude/
    └── skills/
        ├── ski-research/SKILL.md
        ├── ski-flow/SKILL.md
        ├── ski-env/SKILL.md
        ├── ski-dev/SKILL.md
        ├── ski-test/SKILL.md
        └── ski-docs/SKILL.md
```

Runtime output goes into `.ski/` inside the user's working project — never into the project root itself:

```
<user-project>/
└── .ski/
    ├── ski-research/
    │   ├── research.md      ← research report
    │   └── poc/             ← proof-of-concept scripts and results
    ├── ski-flow/            ← execution plan artifacts
    ├── ski-env/             ← scaffold reports
    ├── ski-dev/             ← implementation logs
    ├── ski-test/            ← test reports
    └── ski-docs/            ← generated documentation
```

The `.ski/` directory is the working space. The user's project files are never touched by skibuild modules directly.

## Downstream ecosystem

- **skibundle** — packages exported skills for distribution
- **skibench** — benchmarks skill performance
