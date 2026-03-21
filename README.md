# skibuild

skibuild is the core engine of the ski.ai ecosystem, designed to orchestrate the lifecycle of custom AI "skills." It provides a CLI-driven framework for transitioning an AI tool from a conceptual Plan (research and flow design) through Implementation (development and testing) to a standardized Export (documentation and metadata).

## Core workflows

- `ski init`: from-scratch workflow for generating new skill scaffolding.
- `ski refine`: iterative workflow for updating and expanding existing skills.

## Modules

- `ski-research`: document parsing, internet fetching, and component analysis.
- `ski-flow`: command grouping, argument parsing, and flow state management.
- `ski-env`: git initialization, folder structuring, and boilerplate generation.
- `ski-dev`: runtime environment for implementation and integration.
- `ski-test`: type-checking and entry-point validation.
- `ski-docs`: generators for `SKILL.md`, `README.md`, and `CLAUDE.md`.

## Repository goal

Provide a seamless, reproducible environment where AI-native tools can be built, verified, and exported as standardized skills for use in skilib and benchmarking in skibench.
