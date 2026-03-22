# ski-dev

## Purpose

Run implementation-time automation for developing and iterating on skill logic.

## Responsibilities

- Execute development entry points.
- Coordinate runtime hooks and local tooling.
- Validate module wiring and command behavior.
- Support iterative refine loops.

## Inputs

- Build plan from `ski-flow`.
- Current project source tree.
- Runtime config and local environment.

## Outputs

- Implementation execution logs.
- Updated generated code and integration glue.
- Developer-facing diagnostics.

## Quality checks

- Entry points resolve successfully.
- Runtime errors are grouped and actionable.
- Repeat runs remain deterministic where expected.
