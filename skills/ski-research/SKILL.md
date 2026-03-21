# ski-research

## Purpose

Provide the research layer for skibuild by collecting and normalizing source information used during planning and refinement.

## Responsibilities

- Parse local docs and structured files.
- Fetch and summarize web resources.
- Analyze components and dependency context.
- Produce normalized research artifacts for downstream flow planning.

## Inputs

- User intent and skill objective.
- URLs, markdown/docs, code paths.
- Optional existing skill metadata.

## Outputs

- Structured research notes.
- Source registry with links and relevance scores.
- Gaps, risks, and assumptions list.

## Quality checks

- Sources are traceable and deduplicated.
- Findings map to objective and constraints.
- Unknowns are explicit.
