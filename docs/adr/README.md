# ADR Index

This directory holds short Architecture Decision Records derived from the foundation document.

The goal is to make the most important architecture choices easy to find without re-reading all of:

- [foundation.md](/Users/user/code/pi-mirror/docs/foundation.md)

## Accepted ADRs

- [ADR-0001: Runtime vs Executor Separation](/Users/user/code/pi-mirror/docs/adr/0001-runtime-vs-executor-separation.md)
- [ADR-0002: Named Workspace Model](/Users/user/code/pi-mirror/docs/adr/0002-named-workspace-model.md)
- [ADR-0003: Staging vs Canonical Storage Model](/Users/user/code/pi-mirror/docs/adr/0003-staging-vs-canonical-storage.md)
- [ADR-0004: Profile and Tool Authorization Model](/Users/user/code/pi-mirror/docs/adr/0004-profile-and-tool-authorization.md)
- [ADR-0005: Instance Directory and Config Strategy](/Users/user/code/pi-mirror/docs/adr/0005-instance-directory-and-config.md)

## Next ADR Candidates

- local execution backend choice for the first execution-capable release
- VM-class executor for production
- approval service abstraction
- agent preset model
- promotion workflow boundary
- audit and event logging strategy

## Suggested Handoff Sequence

For a fresh coding agent session:

1. read [foundation.md](/Users/user/code/pi-mirror/docs/foundation.md)
2. read ADR-0001 through ADR-0005
3. define shared interfaces in `apps/dev-cli/src/domain`
4. implement config loading in `apps/dev-cli/src/config`
5. only then start runtime wiring in `apps/dev-cli/src/runtime`
