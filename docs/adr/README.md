# ADR Backlog

This directory will hold short Architecture Decision Records derived from the foundation document.

The goal is to extract the highest-value decisions from:

- [foundation.md](/Users/user/code/pi-mirror/docs/foundation.md)

so a fresh coding agent can work from smaller, stable decisions instead of re-parsing the entire narrative every time.

## Recommended ADR Order

### ADR-0001: Runtime vs Executor Separation

Decision:

- Pi runtime and code execution are separate concerns
- the runtime owns sessions, orchestration, and tool presentation
- the executor owns command execution and file mutation

Why first:

- this is the highest-leverage long-lived boundary

### ADR-0002: Named Workspace Model

Decision:

- the system reasons about named workspaces instead of raw host paths
- policies and logs should refer to workspace ids and kinds

Why second:

- this shapes config, policy, and executor interfaces

### ADR-0003: Staging vs Canonical Storage Model

Decision:

- each session gets one writable staging workspace
- canonical and reference stores remain read-only in v1
- promotion into canonical stores is a future workflow

Why third:

- this is the core data-safety boundary

### ADR-0004: Profile And Tool Authorization Model

Decision:

- profiles are fixed at session start
- profiles are tool-first
- default profile does not include shell access
- safety comes from guardrails first, approvals second

Why fourth:

- this shapes policy and tool wiring

### ADR-0005: Instance Directory And Config Strategy

Decision:

- personal config and runtime data live in `.instance/` by default during development
- repo contains code, schemas, defaults, and examples
- production should support an external instance dir

Why fifth:

- this shapes configuration and deployment early without blocking core runtime work

## Candidate Later ADRs

- Docker executor for development
- VM-class executor for production
- approval service abstraction
- agent preset model
- promotion workflow boundary
- audit and event logging strategy

## Suggested Handoff Sequence

For a fresh coding agent session:

1. read [foundation.md](/Users/user/code/pi-mirror/docs/foundation.md)
2. write ADR-0001 through ADR-0005
3. define shared interfaces in `packages/core`
4. implement config loading in `packages/config`
5. only then start runtime wiring in `packages/pi-runtime`
