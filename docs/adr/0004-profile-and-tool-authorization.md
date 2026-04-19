# ADR-0004: Profile and Tool Authorization Model

Status: Accepted

## Context

`pi-mirror` needs a permission model that is understandable, durable, and not dominated by noisy confirmation prompts.

The system also needs room for future profiles with different trust envelopes, such as read-only, coding, or curator-style behaviors.

## Decision

Profiles are fixed at session start and are tool-first.

Each profile defines:

- readable workspaces
- one writable workspace, if any
- allowed tools
- sandbox settings
- approval and policy rules

Primary safety comes from hard guardrails:

- workspace boundaries
- read-only canonical/reference stores
- executor isolation
- narrow writable surfaces

Approvals are a second layer for higher-risk actions, not the main safety model.

The default profile does not include shell access.

Default direction:

- `read`, `ls`, `grep`, `find`

More privileged profile direction:

- `read`, `ls`, `grep`, `find`, `edit`, `write`, `bash`

## Consequences

Positive:

- clearer operator model
- less approval noise
- safer default posture
- clean path to future narrower or broader profiles

Trade-offs:

- profile escalation should happen through a new session or explicit handoff, not silent in-place mutation
- the host must own the tool contract instead of inheriting whatever the runtime happens to expose by default

## Alternatives Considered

- capability-only profiles
  Rejected as the main external model because tool-first configuration is easier to understand and operate.
- permission-first UX for routine actions
  Rejected because it becomes noisy and shifts too much burden onto approvals.

## Notes

- Internal policy may still normalize actions into a shallow set such as `fs.read`, `fs.write`, and `exec.shell`.
- The external permission surface remains tool-first.
