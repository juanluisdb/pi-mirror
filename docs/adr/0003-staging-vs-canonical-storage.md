# ADR-0003: Staging vs Canonical Storage Model

Status: Accepted

## Context

`pi-mirror` is meant to operate around personal files and personal knowledge stores. The system needs a safe place for agent-generated work without contaminating trusted personal stores.

This is especially important for note vaults, documents, and any knowledge base meant to reflect the user's own thinking.

## Decision

For v1, each session gets:

- one writable staging workspace
- zero or more readable workspaces

Canonical and reference stores remain read-only to the agent.

Agent-generated artifacts must land in staging first. Promotion into canonical stores is a future workflow and is out of scope for the first implementation.

## Consequences

Positive:

- protects trusted personal stores
- keeps provenance cleaner
- makes policy and enforcement simpler
- matches the product goal of a safe, controlled agent environment

Trade-offs:

- some cross-store workflows stay manual at first
- direct editing of canonical content is intentionally unavailable in early releases

## Alternatives Considered

- direct agent writes into canonical stores
  Rejected for v1 because contamination risk and weak provenance are too high.
- many writable workspaces in one session
  Rejected for v1 because it makes policy, audits, and backend behavior harder to reason about.

## Notes

- Promotion is expected to become a later system feature owned by the product shell, not an ad hoc prompt behavior.
- The staging/canonical distinction is a product concept, not a backend implementation detail.
