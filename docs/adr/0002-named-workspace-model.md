# ADR-0002: Named Workspace Model

Status: Accepted

## Context

`pi-mirror` needs to expose files to the agent while keeping policy, audit, and future backend changes understandable.

Using raw host paths as the main product model would couple policy to machine-specific details and make future backend changes harder to reason about.

## Decision

The system reasons about named workspaces, not raw host paths.

Examples:

- `staging`
- `personal_docs`
- `vault`

Policies, logs, and session context should refer to:

- workspace id
- workspace kind
- intended access mode

The main workspace kinds are:

- `staging`
- `reference`
- `canonical`

The implementation may resolve these to concrete host paths, mounts, worktrees, overlays, or VM-visible paths, but those are backend details.

## Consequences

Positive:

- cleaner operator mental model
- safer and more portable policy rules
- better logs and audit semantics
- easier future promotion workflows

Trade-offs:

- requires a workspace registry instead of directly passing file paths around
- config has one more abstraction layer than a simple path list

## Alternatives Considered

- raw host paths in config, policy, and UX
  Rejected because they are harder to audit, harder to migrate, and too tied to one machine layout.

## Notes

- Workspaces may later be resolved through different execution backends.
- Backend-specific mechanisms such as worktrees or shadow copies do not replace the named workspace model.
