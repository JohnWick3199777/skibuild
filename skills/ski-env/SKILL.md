# ski-env

## Purpose

Automate project environment setup so each generated or refined skill starts from a reproducible baseline.

## Responsibilities

- Initialize git and repository conventions.
- Create folder structures and templates.
- Generate baseline config and boilerplate.
- Prepare environment variables and local defaults.

## Inputs

- Execution plan from `ski-flow`.
- Target skill name and module selections.
- Optional template profile.

## Outputs

- Scaffolded project tree.
- Bootstrap configuration files.
- Setup report with actions performed.

## Quality checks

- Generated structure matches expected template.
- Idempotent reruns avoid destructive changes.
- Missing prerequisites are surfaced with clear errors.
