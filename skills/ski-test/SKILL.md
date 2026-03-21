# ski-test

## Purpose

Provide verification gates for generated and refined skills before export.

## Responsibilities

- Run type checks.
- Validate CLI entry points.
- Execute smoke and contract tests.
- Publish pass/fail status for pipeline continuation.

## Inputs

- Implementation artifacts from `ski-dev`.
- Test configuration and fixture data.
- Optional strictness/profile flags.

## Outputs

- Test reports and summaries.
- Structured failure diagnostics.
- Verification status for export readiness.

## Quality checks

- Failing checks block export.
- Reports identify failing stage and command.
- Results are machine-readable and human-readable.
