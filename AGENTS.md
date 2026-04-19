# pi-mirror Agent Guide

- Put durable design decisions in [docs/foundation.md](/Users/user/code/pi-mirror/docs/foundation.md) or `docs/adr/`.
- Official runtime is Node. Prefer the repo-pinned version once `.nvmrc` or equivalent exists.
- Use `pnpm` for package and workspace management.
- Build `pi-mirror` as a product shell around embedded Pi, not as a wrapper around the Pi CLI.
- Use GitHub issues as the execution layer. Keep durable design in docs and ADRs.
- Prefer one parent issue for an initiative, then smaller sub-issues with explicit dependencies when the work can be split.
- Prefer thin, independently grabbable issue slices over one giant issue for a whole release.
- When implementing an issue, read linked ADRs/RFCs first and stay within the issue scope.
- Keep personal mappings and machine-specific paths out of repo code and docs; use `.instance/` config or examples instead.
- Keep observability choices OpenTelemetry-friendly: bootstrap at app entrypoints, keep package code vendor-neutral, and never put prompts, file contents, secrets, or raw command output into trace attributes.
