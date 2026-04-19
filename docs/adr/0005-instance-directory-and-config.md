# ADR-0005: Instance Directory and Config Strategy

Status: Accepted

## Context

`pi-mirror` needs to remain portable as a repo while also supporting personal paths, personal runtime data, and machine-specific configuration.

Committing personal mappings into the repo would make the project harder to share, harder to back up cleanly, and more likely to leak local details.

## Decision

Development uses a local instance directory:

- `.instance/`

The instance directory holds personal and machine-specific state such as:

- config
- runtime data
- logs
- staging data
- secret files or secret references

The repo holds:

- code
- schemas
- examples
- defaults
- documentation

Configuration is file-driven and validated by the app. The expected config surface starts with:

- `workspaces.yaml`
- `profiles.yaml`
- `agents.yaml`

Production should support an explicit external instance directory later.

## Consequences

Positive:

- repo stays portable and mostly agnostic
- personal paths stay out of version control
- backup boundaries are clearer
- local development and later deployment share the same mental model

Trade-offs:

- config loading needs explicit instance-dir resolution
- the first CLI needs to bootstrap around external state instead of assuming everything lives in repo code

## Alternatives Considered

- keep personal config in tracked repo files
  Rejected because it makes the repo less portable and more likely to capture personal details.
- treat runtime state as ad hoc app data outside an explicit instance model
  Rejected because it weakens backup, deployment, and operator clarity.

## Notes

- `.instance/` is the development default, not a claim that production must use the same physical location.
- The repo should provide examples and schemas, not personal mappings.
