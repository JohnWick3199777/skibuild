# ski-flow

## Purpose

Define and coordinate command flow design from research output into executable CLI pipelines.

## Responsibilities

- Group commands by workflow stage.
- Parse and validate arguments/options.
- Track flow state transitions (plan, implement, export).
- Enforce deterministic step ordering.

## Inputs

- Research artifacts from `ski-research`.
- User-selected command and flags.
- Existing project state.

## Outputs

- Resolved execution plan.
- Stage-by-stage state snapshot.
- Actionable handoff payloads for env/dev/test/doc modules.

## Quality checks

- Invalid arg combinations are rejected early.
- Flow transitions are explicit and logged.
- Dry-run mode produces stable plans.
