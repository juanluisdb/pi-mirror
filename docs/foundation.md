# pi-mirror Foundation

Date: 2026-04-12
Status: Draft foundation record

## What This Document Is

This document is the durable design record for the initial `pi-mirror` architecture work.

It is intentionally longer than a normal README note. The goal is to preserve:

- what we are trying to build
- the important decisions we made
- the alternatives we considered
- the reasoning behind those decisions
- the Pi-specific and sandbox-specific findings that informed them
- the current roadmap and open questions
- the sources that were read, inspected, or referenced

This is not an API spec and not an implementation plan down to the class level. It is the high-signal architectural foundation the repo should be able to grow from.

## Project Intent

`pi-mirror` is a personal AI agent system built on top of Pi. It is intended to run on personal infrastructure, starting locally during development and later on a home server.

The target system should eventually support:

- persistent agent sessions and memory
- internet access when allowed
- access to selected personal files
- code execution in an isolated environment
- multiple profiles with different safety and capability envelopes
- future external interfaces such as Telegram or SSH

The system should not start as a generalized multi-tenant agent platform. It is a single-user personal system first.

## Primary Goal

Build a single-user personal agent that can:

- run on personal infrastructure
- interact with files and code in controlled workspaces
- access the internet when allowed
- execute code in an isolated environment
- keep durable session state and memory
- evolve later into a multi-profile and eventually multi-agent system without a rewrite

## Design Priorities

The system is optimized for:

- maintainability
- safety by architecture, not by prompt wording
- clear long-lived boundaries
- incremental growth without a rewrite
- human-auditable behavior
- local operability

It is not optimized for:

- fastest possible prototype at any cost
- maximizing autonomy before boundaries exist
- early multi-agent orchestration
- early channel integration

## Working Assumptions

Current assumptions:

- there is one primary user
- development starts locally on the laptop
- later deployment target is a personal home server
- the system may eventually have internet access and access to personal files
- some personal stores should be readable by the agent
- direct writes into canonical personal stores should be avoided in v1
- sync interactive approvals are fine for v1
- async approvals are a future requirement, not a v1 requirement

## High-Level Architectural Thesis

The most important architectural idea is:

- Pi is the runtime kernel
- `pi-mirror` is the product shell around it

And the most important security and operations idea is:

- runtime and execution are separate concerns

Everything else in the foundation mostly follows from those two choices.

## Why Pi

Pi is a strong fit because it already has:

- a session model
- session persistence
- compaction
- a tool runtime
- an extension system
- SDK embedding and RPC modes
- a clear layered architecture

The design direction from the Pi references is consistent:

- let Pi own the agent runtime, context assembly, session behavior, and resource loading
- let the host app own identity, policy, delivery semantics, sandboxing, and higher-level orchestration

This made Pi a better fit than trying to invent a fresh agent kernel from scratch.

## Pi Findings That Shaped The Design

### 1. Pi Is Layered On Purpose

The relevant layers are:

- `pi-ai`
- `pi-agent-core`
- `pi-coding-agent`
- product shell

The right layer for `pi-mirror` is `pi-coding-agent`, embedded through the SDK, with a host-owned product shell around it.

Why this matters:

- it means we do not have to reinvent sessions, compaction, or the basic runtime
- it also means we should not force Pi to own concerns that belong to the product shell, such as delivery channels, personal policy, or deployment

### 2. Pi Supports Host Embedding Better Than A Thin CLI Wrapper

The Pi references consistently point toward SDK embedding or RPC for serious integrations.

That matters because:

- a wrapper around the CLI would leak too many local assumptions
- a host-owned integration gives us much cleaner long-lived seams

### 3. Pi Built-In Tools Are Flexible, But Not Fully "Remote By Magic"

This was an important finding.

The built-in tools can be created for a custom `cwd`, and several tools expose custom operations:

- `bash` supports custom execution operations and a spawn hook
- `read`, `edit`, `ls`, and `find` expose pluggable operations

However, some tools still assume local execution details in practice:

- `grep` still shells out to local `rg` by default
- some search and filesystem assumptions are easier to reason about if the host owns the canonical tool layer

This led to the recommendation that the host should own the real tool contract and only expose stable logical tool names to Pi.

### 4. Pi Extensions Are Strong Enough For Policy Hooks

Pi's extension system can:

- register tools
- intercept tool calls
- observe tool results
- inject UI
- implement approval or blocking logic

That means we do not need to fork Pi to implement policy, approval, or extra tool behavior.

### 5. Pi Session Semantics Matter

Pi's session model is richer than "just a chat log." That matters for future profile handoffs, session branching, and eventual multi-profile coordination.

This influenced several decisions:

- profile should be fixed at session start
- escalation should be explicit
- handoff should eventually be a first-class concept rather than a silent permission change in the middle of a session

## Core Design Decisions

### 1. Pi Is The Runtime Kernel, Not The Whole Product

Pi should own:

- agent runtime
- sessions
- compaction
- model interaction
- extension and tool integration surface

`pi-mirror` should own:

- workspace registry
- policy and approvals
- executor boundary
- deployment
- instance config
- future channel integrations
- future promotion workflows

Reason:

- keeps Pi in the role it is good at
- avoids forcing the entire product into Pi's local CLI assumptions
- makes the system easier to maintain and evolve

### 2. Runtime And Execution Are Separate Concerns

There are two planes:

- `runtime`: long-lived Pi session, memory, orchestration, policy, and tool presentation
- `executor`: where commands and file mutations actually happen

Reason:

- safer than letting arbitrary generated code run next to long-lived session state and secrets
- easier to swap Docker for a VM-backed executor later
- easier to isolate failures
- easier to audit
- clearer mental model

### 3. Development Uses Docker; Production Should Use A VM-Class Boundary

Current plan:

- local development: Docker-backed executor
- later home-server deployment: VM-class execution boundary

Reason:

- Docker is fast and convenient for local iteration
- plain containers are not trusted enough as the only security boundary for internet-enabled arbitrary code execution on a personal home server
- the execution backend should be replaceable, so this can be an implementation migration rather than an architectural rewrite

### 4. Named Workspaces Instead Of Raw Paths

The system should think in terms of named workspaces such as:

- `staging`
- `personal_docs`
- `vault`

Not in terms of raw host paths.

Reason:

- clearer user and operator mental model
- cleaner policy model
- better logs and audits
- easier backend portability
- easier future promotion workflows

### 5. One Writable Staging Workspace Per Session

For v1, a session gets:

- one primary writable workspace
- zero or more readable workspaces

Canonical personal stores remain read-only.

Reason:

- safer default
- easier to reason about
- better match with Pi's working-directory model
- avoids contamination of personal stores
- simplifies backend and policy enforcement

### 6. Canonical Stores Are Read-Only In V1

Agent-generated artifacts should land in staging first. Promotion into canonical stores is a future system feature, but not a v1 implementation task.

Reason:

- preserves quality and provenance of personal knowledge
- avoids mixing agent-generated drafts into trusted personal stores
- keeps v1 smaller and safer

### 7. Safety Comes From Hard Guardrails First, Approvals Second

Primary safety should come from:

- workspace boundaries
- executor isolation
- read-only mounts for canonical/reference stores
- no host Docker socket exposure
- narrow writable surfaces
- minimal secret exposure

Approvals are still useful, but should be reserved for exceptional or high-risk actions.

Reason:

- avoids a noisy approval UX
- makes autonomy possible inside safe boundaries
- does not rely on prompts as the main enforcement layer
- leads to a more trustworthy system

### 8. Profiles Are Tool-First And Fixed At Session Start

Profiles should define:

- readable and writable workspaces
- allowed tools
- sandbox settings
- approval rules

Profiles should not change silently mid-session.

Reason:

- clear audit trail
- easier mental model
- better future handoff/escalation behavior
- easier long-term maintenance

### 9. Default Profile Should Not Have Shell Access

Default profile direction:

- structured tools only: `read`, `ls`, `grep`, `find`

More privileged profile direction:

- coding profile: `read`, `ls`, `grep`, `find`, `edit`, `write`, `bash`

Reason:

- shell access is qualitatively riskier than structured tools
- better default trust posture
- cleaner future separation between research, coding, and curator-style profiles

### 10. Personal Config Lives In An Instance Directory, Not In Git

The repo should be portable and mostly agnostic. Personal mappings and runtime data live in `.instance/` during development, with support for an external instance dir later.

Reason:

- avoids committing personal paths and preferences
- keeps the repo reusable
- makes backup strategy explicit

## Alternatives Considered

### Alternative A: Let Pi CLI Be The Product

Rejected in favor of SDK embedding.

Why it was tempting:

- less code to write
- faster initial experiment

Why it was rejected:

- poor long-term control over policy and executor boundaries
- awkward fit for named workspaces and future profile orchestration
- too many local assumptions leak into the product

### Alternative B: No Runtime/Executor Separation

Rejected.

Why it was tempting:

- simplest thing to build
- fewer moving pieces

Why it was rejected:

- weak safety model
- runtime and arbitrary code execution would share the same failure domain
- later migration to stronger isolation would be much more painful

### Alternative C: Docker As The Permanent Security Boundary

Rejected for production.

Why it was tempting:

- easiest path operationally
- good enough for development

Why it was rejected:

- containers share the host kernel
- the project is meant to run arbitrary generated code with internet access
- the target environment contains personal files

### Alternative D: Raw Host Paths In Policy And UX

Rejected.

Why it was tempting:

- simple at first glance
- maps directly to the filesystem

Why it was rejected:

- hard to audit
- harder to migrate backends
- policy would become coupled to machine-specific details

### Alternative E: Permission-First UX Everywhere

Rejected.

Why it was tempting:

- sounds safe
- resembles common coding-agent products

Why it was rejected:

- too noisy
- pushes too much burden onto approvals
- the better design is to make routine actions safe by construction

### Alternative F: Direct Agent Writes Into Canonical Stores

Rejected for v1.

Why it was tempting:

- more magical workflow
- fewer manual steps later

Why it was rejected:

- high contamination risk
- weak provenance
- mixes agent-generated drafts into trusted personal spaces too early

## The Staging vs Canonical Model

One of the most important conceptual decisions is the separation between:

- canonical stores
- staging stores

Canonical stores are:

- human-authored or human-curated
- high-signal
- intended to remain trusted

Staging stores are:

- agent-writable
- draft-oriented
- allowed to be messy
- the place where artifacts should be created first

This is directly aligned with the idea of keeping a personal knowledge base or vault clean while allowing an agent to generate rough material elsewhere first.

For v1:

- the system enforces the separation
- promotion across the boundary is manual and external to the system

For later:

- the system may gain explicit promotion workflows
- the system may track provenance and approval state for promoted artifacts

## Guardrails vs Permissions

Another key design direction is:

- guardrails first
- permissions second

Guardrails are architectural and non-negotiable:

- no writes outside writable workspace
- canonical stores mounted read-only
- executor boundary for dangerous work
- no host Docker socket exposure
- minimal secret exposure

Permissions and approvals exist for edge cases:

- destructive shell commands
- package installs
- long-running jobs
- future promotion into canonical stores
- future external side effects

This is meant to produce a better autonomy model than "ask for everything all the time."

## Tool Surface Philosophy

The human-facing permission surface should be tool-first:

- `read`
- `ls`
- `grep`
- `find`
- `edit`
- `write`
- `bash`

This is easier to understand and aligns with how people think about what an agent can do.

Internally, policy may still use a small normalized action set:

- `fs.read`
- `fs.search`
- `fs.write`
- `exec.shell`
- `promote.write`

But the normalization should stay shallow and focused on safety-relevant actions only.

The system should not try to build a giant abstract semantic policy engine up front.

## Why Tool-First Profiles Won

We considered capability-only profiles, but tool-first profiles fit better because:

- they are more obvious to operators
- they match actual agent affordances
- they are closer to the shape of Pi tools
- they are easier to evolve with custom tools later

The recommended model is:

- external config stays tool-first
- internal implementation may group tools into helper capability classes when useful

## Profiles, Agents, And Future Coordination

The long-term direction is not "one super-agent with every permission."

The preferred direction is:

- multiple profiles with narrow authority surfaces
- later, possibly multiple agent presets built on those profiles
- later, explicit handoff or coordination between profiles

This suggests a future model like:

- `default`: structured read/search tools only
- `coding`: staging writes plus shell access
- `curator`: narrow read access to staging and canonical stores, no arbitrary shell, maybe future promotion authority

This is safer and cleaner than one broad "master agent."

## Current Repo Shape

```text
pi-mirror/
  apps/
    dev-cli/
  packages/
    core/
    config/
    policy/
    executor-docker/
    pi-runtime/
  docs/
    adr/
  config/
    examples/
    schemas/
  .instance/
    config/
    data/
    secrets/
```

## Package Responsibilities

### `apps/dev-cli`

Local developer entrypoint for:

- selecting an agent preset or profile
- launching a session
- running sync approval flows during development

### `packages/core`

Shared domain concepts and interfaces:

- workspace ids and metadata
- profile ids and agent preset ids
- executor interface
- approval request and decision types
- normalized action types used internally by policy

### `packages/config`

Loading and validating instance config such as:

- `workspaces.yaml`
- `profiles.yaml`
- `agents.yaml`

This package should own:

- schema validation
- default merging
- resolution of instance-relative paths

### `packages/policy`

Policy evaluation and guardrails:

- writable workspace enforcement
- tool authorization
- risk classification
- approval requests
- path and workspace-kind restrictions

### `packages/executor-docker`

Development executor backend using Docker.

This package should be treated as:

- a backend implementation
- not the canonical security model for production

### `packages/pi-runtime`

Pi embedding plus host-owned tool registration.

Important rule:

- Pi should see stable logical tool names
- the host owns what those tools actually do

This package is where the design should prevent local filesystem assumptions from leaking directly into the product.

## Draft Interface Sketches

These are not final APIs. They are early shape sketches intended to preserve the architectural seams we want before implementation details start to leak across package boundaries.

### Core Domain Types

```ts
export type WorkspaceId = string;
export type ProfileId = string;
export type AgentPresetId = string;

export type WorkspaceKind = "staging" | "reference" | "canonical";
export type WorkspaceAccess = "ro" | "rw";

export interface WorkspaceDefinition {
  id: WorkspaceId;
  kind: WorkspaceKind;
  path: string;
  access: WorkspaceAccess;
}

export interface ProfileDefinition {
  id: ProfileId;
  readableWorkspaces: WorkspaceId[];
  writableWorkspace?: WorkspaceId;
  tools: {
    allow: string[];
    ask?: string[];
    deny?: string[];
  };
  sandbox: {
    network: "enabled" | "disabled";
    executor: string;
  };
  rules?: PolicyRule[];
}

export interface AgentPresetDefinition {
  id: AgentPresetId;
  profile: ProfileId;
  model?: string;
  thinking?: "off" | "minimal" | "low" | "medium" | "high" | "xhigh";
  memoryScope?: string;
}
```

### Workspace Registry

```ts
export interface ResolvedWorkspace {
  id: WorkspaceId;
  kind: WorkspaceKind;
  hostPath: string;
  access: WorkspaceAccess;
}

export interface WorkspaceRegistry {
  list(): Promise<ResolvedWorkspace[]>;
  get(workspaceId: WorkspaceId): Promise<ResolvedWorkspace>;
}
```

### Session Launch And Resolution

```ts
export interface SessionLaunchRequest {
  profileId: ProfileId;
  agentPresetId?: AgentPresetId;
  primaryWorkspaceId?: WorkspaceId;
}

export interface ResolvedSessionContext {
  profile: ProfileDefinition;
  agentPreset?: AgentPresetDefinition;
  readableWorkspaces: ResolvedWorkspace[];
  writableWorkspace?: ResolvedWorkspace;
}
```

### Executor

The executor is the key long-lived seam. It must not leak Docker-specific details into the rest of the application.

```ts
export interface CommandSpec {
  workspaceId: WorkspaceId;
  cwd?: string;
  command: string;
  env?: Record<string, string>;
  timeoutMs?: number;
}

export interface CommandResult {
  exitCode: number | null;
  stdout: string;
  stderr: string;
  truncated?: boolean;
}

export interface ReadFileSpec {
  workspaceId: WorkspaceId;
  path: string;
}

export interface WriteFileSpec {
  workspaceId: WorkspaceId;
  path: string;
  content: string;
}

export interface EditFileSpec {
  workspaceId: WorkspaceId;
  path: string;
  edits: Array<{ oldText: string; newText: string }>;
}

export interface SearchSpec {
  workspaceId: WorkspaceId;
  path?: string;
  pattern: string;
  glob?: string;
  limit?: number;
}

export interface Executor {
  runCommand(spec: CommandSpec): Promise<CommandResult>;
  readFile(spec: ReadFileSpec): Promise<string>;
  writeFile(spec: WriteFileSpec): Promise<void>;
  editFile(spec: EditFileSpec): Promise<void>;
  listFiles(spec: { workspaceId: WorkspaceId; path?: string; limit?: number }): Promise<string[]>;
  findFiles(spec: { workspaceId: WorkspaceId; pattern: string; path?: string; limit?: number }): Promise<string[]>;
  grepFiles(spec: SearchSpec): Promise<string>;
  resetWorkspace(workspaceId: WorkspaceId): Promise<void>;
}
```

### Approval Service

```ts
export interface ApprovalRequest {
  id: string;
  title: string;
  action: ActionRequest;
  reason: string;
}

export interface ApprovalDecision {
  outcome: "allow" | "deny";
  scope?: "once" | "session" | "always";
}

export interface ApprovalService {
  request(input: ApprovalRequest): Promise<ApprovalDecision>;
}
```

### Policy Engine

Policy should evaluate concrete action requests, not raw tool calls everywhere. The normalized action model should stay shallow.

```ts
export type ActionKind =
  | "fs.read"
  | "fs.search"
  | "fs.write"
  | "exec.shell"
  | "promote.write";

export interface ActionRequest {
  kind: ActionKind;
  toolName: string;
  workspaceId: WorkspaceId;
  workspaceKind: WorkspaceKind;
  path?: string;
  command?: string;
  riskTags?: string[];
}

export type PolicyEffect = "allow" | "ask" | "deny";

export interface PolicyRule {
  id: string;
  when: Record<string, unknown>;
  effect: PolicyEffect;
}

export interface PolicyDecision {
  effect: PolicyEffect;
  ruleId?: string;
  reason?: string;
}

export interface PolicyEngine {
  evaluate(action: ActionRequest, ctx: ResolvedSessionContext): Promise<PolicyDecision>;
}
```

### Pi Tool Wiring

The runtime should expose stable logical tools to Pi while delegating real execution to the host-owned services.

```ts
export interface ToolRuntimeContext {
  session: ResolvedSessionContext;
  executor: Executor;
  policy: PolicyEngine;
  approvals: ApprovalService;
}

export interface PiToolFactory {
  createTools(ctx: ToolRuntimeContext): Promise<unknown[]>;
}
```

These interface sketches are intentionally a little boring. That is a good sign. They are trying to preserve durable seams rather than expose clever abstractions.

## Instance Layout

Development default:

```text
.instance/
  config/
  data/
  secrets/
```

Planned later behavior:

- support `AGENT_INSTANCE_DIR` override
- use `.instance/` as dev default
- require explicit external location in production

The instance directory should hold:

- personal config
- runtime state
- logs
- memory stores
- staging data
- secrets or secret file references

The repo should hold:

- code
- schemas
- defaults
- examples
- documentation

## Config Direction

### `workspaces.yaml`

Expected shape:

- named workspace ids
- workspace kind
- path
- intended access mode

Example direction:

```yaml
workspaces:
  staging:
    kind: staging
    path: "./data/staging"
    access: rw

  personal_docs:
    kind: reference
    path: "/abs/path/to/docs"
    access: ro

  vault:
    kind: canonical
    path: "/abs/path/to/vault"
    access: ro
```

Important note:

- `access` in config is declarative intent
- real enforcement still comes from mounts, executor behavior, and policy

### `profiles.yaml`

Expected shape:

- readable workspaces
- one writable workspace
- allowed, asked, or denied tools
- sandbox settings
- rule expressions for risky actions

Example direction:

```yaml
profiles:
  default:
    workspaces:
      readable: [staging, personal_docs]
      writable: staging
    tools:
      allow: [read, ls, grep, find]
      ask: []
      deny: [edit, write, bash]
    sandbox:
      network: enabled
      executor: docker-dev
    rules:
      - id: deny-canonical-writes
        when:
          action: fs.write
          workspace_kind: canonical
        effect: deny
```

### `agents.yaml`

Expected shape:

- named agent preset
- profile selection
- model preference
- thinking level
- memory scope

Important split:

- profile answers "what is allowed?"
- agent preset answers "how should this behave?"

That split is intentionally designed to avoid coupling security posture to model choice.

## Approval Model

v1 approval model:

- synchronous
- local
- terminal driven

The architecture should still assume a generic approval service interface so that later implementations can include:

- Telegram approvals
- web approvals
- queue-backed async approvals

The important design rule is:

- policy should not depend on terminal UI directly

## OpenClaw Comparison And Validation

OpenClaw is not the target architecture for `pi-mirror`, but its docs were still useful for validation because it is one of the more serious examples of Pi used as an embedded runtime inside a larger product shell.

### What OpenClaw Validated

#### 1. Embedding Pi Directly Is The Right Direction

OpenClaw’s Pi integration doc explicitly says it embeds Pi through `createAgentSession()` instead of spawning Pi as a subprocess or using RPC mode for the main agent path.

That validates our choice to build `pi-mirror` around embedded `pi-coding-agent` rather than around a thin CLI wrapper.

#### 2. The Host Should Own The Tool Contract

One of the strongest confirmations from OpenClaw was its tool split strategy:

- it passes all tools via `customTools`
- it leaves `builtInTools` empty
- it overrides everything so policy and sandboxing remain consistent

This is very close to the direction we already chose for `pi-mirror`.

#### 3. Sandbox Integration Must Touch Tools, Paths, And Prompting Together

OpenClaw’s Pi integration docs treat sandboxing as more than "run exec elsewhere." Their docs describe sandbox-aware tool selection, path constraints, browser bridging, and sandbox information in prompt construction.

This validates the idea that executor isolation, workspace semantics, and runtime tool exposure need to be designed together.

### Approval Lessons From OpenClaw

OpenClaw’s exec approvals model is particularly interesting.

Key ideas:

- approvals are a second-layer safety interlock, not the only policy layer
- effective policy is the stricter of requested tool policy and host-local approvals
- host-local approvals remain the enforceable source of truth
- async approvals store a canonical run plan so later changes to command/cwd/context do not silently alter what gets approved

These are good ideas and worth remembering.

### Ideas Worth Borrowing

- host-owned tool contract instead of trusting stock local Pi tools
- explicit separation between tool policy and host-exec approvals
- canonicalized execution plan for any future async approvals
- operational diagnostics around sandbox state, such as "explain current sandbox" or "recreate sandbox"

### Things Not To Copy Blindly

- OpenClaw’s overall control surface is much broader than what `pi-mirror` needs in v1
- its approval and exec policy layers are powerful, but also more operationally heavy than a single-user local-first system needs at the beginning
- its host exec model is designed around gateway and node hosts, which is useful inspiration but not our simplest first path

### Conclusion

OpenClaw did not change the core plan. It mostly strengthened it:

- embed Pi directly
- own the tools in the host layer
- keep execution policy and runtime policy explicit
- do not rely on raw built-in local tool behavior as the product surface

## Sandbox Model

### Development Sandbox

Development uses Docker because:

- it is fast to bootstrap
- it is good enough for local iteration
- it helps validate the executor boundary early

### Production Sandbox Direction

Production should use a VM-class boundary because:

- the system may eventually execute arbitrary generated code
- the system may have internet access
- the host will contain personal files

This does not commit the project today to a specific production implementation such as:

- Firecracker
- Kata Containers
- a dedicated KVM VM
- another VM-backed executor

But it does commit the architecture to needing a stronger production boundary than plain containers.

## Executor Evolution Strategy

The intended migration path is:

1. define the executor boundary first
2. implement `executor-docker` for development
3. keep runtime code Docker-agnostic
4. later implement a VM-backed production executor

This should be manageable if:

- Docker-specific nouns do not leak into the app surface
- host tooling does not rely on `docker exec` semantics in core packages
- policy reasons about logical actions and workspaces, not container internals

The architecture should specifically avoid:

- exposing `containerId` outside executor packages
- binding policy to Docker mounts
- direct host filesystem access in runtime packages

## Risks And Mitigations

### Risk: Runtime And Executor Boundaries Blur

Mitigation:

- host-owned tool layer
- core interfaces that separate runtime from executor
- avoid stock local tools as the final product surface

### Risk: Docker Assumptions Leak Everywhere

Mitigation:

- keep Docker in backend packages only
- keep workspaces logical
- keep policy backend-agnostic

### Risk: Personal Stores Get Contaminated

Mitigation:

- staging vs canonical split
- canonical stores read-only in v1
- promotion explicitly out of scope for v1

### Risk: Approval UX Becomes Noisy

Mitigation:

- hard guardrails first
- approvals only for risky actions
- default profile without shell access

### Risk: Policy Becomes Overabstracted

Mitigation:

- tool-first external config
- shallow normalized action model internally
- avoid trying to solve every future policy case now

## v1 Scope

### In Scope

- local repo scaffold
- Pi embedding layer
- named workspaces
- profiles
- development CLI
- Docker-backed dev executor
- hard guardrails around writable surfaces
- synchronous approvals in local terminal
- durable local instance directory
- foundational docs and ADRs

### Out Of Scope

- Telegram integration
- web UI
- async approval flows
- canonical-store promotion workflows
- multi-agent orchestration
- production microVM executor implementation

## Roadmap

### Phase 0: Foundation

- create the repo
- create the package layout
- write the foundation document
- write the first ADRs

### Phase 1: Core Shapes

- define shared domain types
- define executor interface
- define config schema and loading
- define profile and workspace resolution

### Phase 2: Policy And Runtime

- implement policy engine with hard guardrails
- embed Pi through the host-owned runtime package
- expose stable logical tools to Pi

### Phase 3: Local Dev Execution

- implement Docker-backed development executor
- support sync terminal approvals
- support instance-local data and config

### Phase 4: Harden And Prepare For Deployment

- improve logs and audit events
- improve instance portability
- prepare production executor abstraction

### Later Phases

- VM-backed production executor
- async approvals
- explicit promotion workflows
- external channel adapters
- multi-profile coordination and handoff

## Phased Implementation And Release Plan

This project is large enough that it needs explicit release slices, not just a vague roadmap.

The goal of this plan is to:

- get to a useful release quickly
- validate the most dangerous architectural seams early
- avoid implementing distant future features too soon

### Release 0: Foundation And Skeleton

Goal:

- create the monorepo
- capture the architecture
- define the main interfaces and boundaries

Deliverables:

- directory scaffold
- foundation document
- first ADR set
- package skeletons

Exit criteria:

- the main boundaries are documented
- future implementation has a clear place to go

### Release 1: Config And Read-Only Runtime

Goal:

- get a minimal local CLI running
- support instance config loading
- support a read-only/default profile

Deliverables:

- `workspaces.yaml`, `profiles.yaml`, `agents.yaml` schema validation
- `.instance` loading
- workspace registry
- dev CLI that can start a session with a default profile
- Pi runtime wired with read-only tools only

Recommended scope:

- `read`
- `ls`
- `grep`
- `find`

No writes, no shell yet.

Why this release matters:

- validates Pi embedding
- validates host-owned tool registration
- validates config model
- keeps safety simple

### Release 2: Staging Writes Without Shell

Goal:

- allow structured file mutation inside the writable staging workspace
- still avoid shell complexity

Deliverables:

- `edit` and `write` support through host-owned tools
- writable staging workspace enforcement
- canonical/reference write denial
- basic audit logging for reads/writes

Why this release matters:

- validates the staging model
- validates hard guardrails
- gets to useful non-shell agent behavior quickly

### Release 3: Docker Executor For Development

Goal:

- introduce the executor boundary for real command execution
- keep it local and development-focused

Deliverables:

- `executor-docker` implementation
- `bash` tool wired through the executor
- coding profile with shell access
- sync terminal approvals for risky shell actions

Why this release matters:

- validates the most important long-term boundary
- gives an actually useful coding-agent development loop
- keeps the production isolation choice open

### Release 4: Hardening And Operator Quality

Goal:

- improve trust and operability before widening scope

Deliverables:

- better risk classification for shell actions
- better audit trails
- better instance diagnostics
- instance backup guidance
- sandbox/executor diagnostics

Potential nice-to-haves:

- explain current workspace bindings
- explain current policy/profile
- explain current executor state

### Release 5: Deployment Baseline

Goal:

- make the system runnable outside the laptop with the same mental model

Deliverables:

- explicit external instance dir support
- deployment docs
- service startup scripts or compose setup
- stronger secrets handling guidance

This release still does not require:

- Telegram
- web UI
- async approvals
- promotion workflows

### Release 6: Safer Production Execution

Goal:

- move beyond development-grade execution isolation

Deliverables:

- VM-backed production executor design and implementation
- production profile guidance
- better network and secrets posture

### Later Releases

Later releases can add:

- async approvals
- promotion bridge workflows
- external channels
- coordinator/curator profiles
- explicit handoff between profiles

## Suggested Immediate Task Sequence

If implementation starts right away, the recommended order is:

1. write ADRs from this foundation
2. define shared interfaces in `packages/core`
3. define config schema and loader in `packages/config`
4. implement workspace registry
5. implement read-only tool wiring in `packages/pi-runtime`
6. build `apps/dev-cli` for launching a session with the default profile
7. add structured write tools for staging
8. only then add Docker executor and shell support

This sequencing keeps the early releases useful while validating the right seams first.

## Open Questions

These are intentionally unresolved. They should be treated as follow-up design work, not as hidden assumptions.

### Unresolved But Not Blocking Release 1

- which VM-backed executor to adopt first for production deployment
- how much network policy to enforce at the executor level in v1 versus later
- what minimal audit trail is worth building in the first implementation
- how far to go with rule expressions before the config becomes too complex
- whether the first production deployment should use one durable VM executor or truly ephemeral execution units

### Additional Follow-Up Topics To Revisit

- secrets strategy:
  env-only, secret files under `.instance`, or a more explicit broker pattern
- memory and persistence design:
  exactly what Pi stores directly versus what `pi-mirror` stores for audit, search, and operations
- backup and restore design for `.instance`
- release acceptance criteria:
  how to define "done" for Release 1 through Release 3 in a way a coding agent can execute against

### Why These Stay Open For Now

- they matter, but they should not delay the first read-only and staging-based releases
- several depend on seeing the first real implementation take shape
- the highest-value architecture decisions are already made and documented above

## Source Record

### Primary Sources Read Or Inspected

- Anthropic: managed agents architecture discussion
  [https://www.anthropic.com/engineering/managed-agents](https://www.anthropic.com/engineering/managed-agents)
- Pi SDK docs
  [https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/docs/sdk.md](https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/docs/sdk.md)
- Pi tools index
  [https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/src/core/tools/index.ts](https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/src/core/tools/index.ts)
- Pi built-in tool implementations:
  [bash.ts](https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/src/core/tools/bash.ts)
  [read.ts](https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/src/core/tools/read.ts)
  [grep.ts](https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/src/core/tools/grep.ts)
  [find.ts](https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/src/core/tools/find.ts)
  [ls.ts](https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/src/core/tools/ls.ts)
  [edit.ts](https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/src/core/tools/edit.ts)
- Pi tutorial gist by Nader Dabit
  [https://gist.githubusercontent.com/dabit3/e97dbfe71298b1df4d36542aceb5f158/raw/213be5313e8e74470eb40333944bc829e2f6df5f/pi_tutorial.md](https://gist.githubusercontent.com/dabit3/e97dbfe71298b1df4d36542aceb5f158/raw/213be5313e8e74470eb40333944bc829e2f6df5f/pi_tutorial.md)
- Firecracker project
  [https://github.com/firecracker-microvm/firecracker](https://github.com/firecracker-microvm/firecracker)
  [https://firecracker-microvm.github.io/](https://firecracker-microvm.github.io/)
- Docker security docs
  [https://docs.docker.com/engine/security/](https://docs.docker.com/engine/security/)
  [https://docs.docker.com/engine/security/seccomp/](https://docs.docker.com/engine/security/seccomp/)
  [https://docs.docker.com/engine/security/rootless/](https://docs.docker.com/engine/security/rootless/)
- Docker AI sandboxes docs
  [https://docs.docker.com/ai/sandboxes/](https://docs.docker.com/ai/sandboxes/)
  [https://docs.docker.com/ai/sandboxes/architecture/](https://docs.docker.com/ai/sandboxes/architecture/)

### Local References Read

- Build-on-Pi references:
  [architecture.md](/Users/user/code/agent-skills/build-on-pi/references/architecture.md)
  [build-patterns.md](/Users/user/code/agent-skills/build-on-pi/references/build-patterns.md)
  [sources.md](/Users/user/code/agent-skills/build-on-pi/references/sources.md)
- Pi book references:
  [ch26-rpc.md](/Users/user/code/pi-book/src/ch26-rpc.md)
  [ch28-mom-slack.md](/Users/user/code/pi-book/src/ch28-mom-slack.md)
  [ch32-boundaries.md](/Users/user/code/pi-book/src/ch32-boundaries.md)
  [ch03-reading-map.md](/Users/user/code/pi-book/src/ch03-reading-map.md)

### Ecosystem References Reviewed

- Awesome Pi Agent list
  [README.md](/tmp/awesome-pi-agent/README.md)
- Concrete security extension example
  [https://github.com/michalvavra/agents/blob/main/agents/pi/extensions/security.ts](https://github.com/michalvavra/agents/blob/main/agents/pi/extensions/security.ts)

Important ecosystem notes:

- there are relevant examples for permission hooks, tool auditing, remote execution, and sandboxing
- there does not appear to be one shared ecosystem-standard permission schema worth copying blindly

### Contextual References Mentioned During Design

- Karpathy Obsidian and LLM wiki note
  [https://gist.githubusercontent.com/karpathy/442a6bf555914893e9891c11519de94f/raw/ac46de1ad27f92b28ac95459c782c07f6b8c964a/llm-wiki.md](https://gist.githubusercontent.com/karpathy/442a6bf555914893e9891c11519de94f/raw/ac46de1ad27f92b28ac95459c782c07f6b8c964a/llm-wiki.md)
- User-shared articles and discussions around Docker vs microVMs:
  [https://huggingface.co/blog/agentbox-master/firecracker-vs-docker-tech-boundary](https://huggingface.co/blog/agentbox-master/firecracker-vs-docker-tech-boundary)
  [https://andrewlock.net/running-ai-agents-safely-in-a-microvm-using-docker-sandbox/](https://andrewlock.net/running-ai-agents-safely-in-a-microvm-using-docker-sandbox/)
  [https://www.docker.com/blog/building-ai-teams-docker-sandboxes-agent/](https://www.docker.com/blog/building-ai-teams-docker-sandboxes-agent/)

## Recommended Next Step

Write the first ADRs capturing the most important boundaries before implementation starts:

1. runtime vs executor separation
2. named workspace model
3. staging vs canonical storage model
4. profile and tool authorization model
5. instance directory and config strategy

After that, define the minimal TypeScript interfaces for:

- executor
- config loader
- workspace registry
- approval service
- policy engine

For handoff to a fresh coding agent, the recommended execution order is:

1. write the ADRs in the order listed above
2. create type definitions in `packages/core` that reflect those ADRs
3. implement config parsing and validation in `packages/config`
4. implement the workspace registry and session resolution
5. wire a read-only Pi runtime before adding writes or shell support
