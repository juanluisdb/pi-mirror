# ADR-0001: Runtime vs Executor Separation

Status: Accepted

## Context

`pi-mirror` is a product shell around embedded Pi. It needs to preserve long-lived session state, policy, and tool presentation while also supporting code execution and file mutation.

If runtime state and arbitrary execution share the same environment, failures and risky actions affect the whole system at once. That makes later isolation work much harder.

This ADR defines the primary architecture boundary. It does not decide the final execution backend.

## Decision

`pi-mirror` separates the system into two concerns:

- `runtime`: Pi session lifecycle, orchestration, tool presentation, approvals, and policy
- `executor`: command execution and file mutation

The runtime is the product-facing authority. The executor is a backend implementation behind it.

The runtime must not depend on backend-specific concepts such as:

- Docker container ids
- VM mount paths
- `gondolin` internals
- direct host filesystem access as the default tool path

The product vocabulary stays:

- workspace
- profile
- policy
- executor

## Consequences

Positive:

- stronger safety boundary
- clearer failure domains
- easier future migration from one execution backend to another
- cleaner audit and policy model

Trade-offs:

- extra integration work up front
- tool wiring has to be host-owned instead of trusting stock local behavior
- some implementations may start with a thinner adapter over Pi `XOperations` before a fuller executor package exists

## Alternatives Considered

- let Pi CLI be the product
  Rejected because it weakens control over policy and execution boundaries.
- keep runtime and execution in one environment
  Rejected because it creates a weak safety model and a painful future migration path.

## Notes

- A Docker-backed development executor is a valid initial backend.
- A VM-backed fast path using Pi `XOperations` is also valid if the product boundary remains intact.
