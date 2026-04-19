# Pi Sandbox Lab Roadmap

This is a learning challenge before committing more implementation to `pi-mirror`.

The goal is to learn enough about Pi, TypeScript, RPC, tools, and Gondolin to make better architecture decisions later. This is intentionally not the product implementation.

## Rules Of The Challenge

- Build a disposable experiment, not production code.
- Use fake files and toy workspaces only.
- Do not mount personal files.
- Do not design the long-term `pi-mirror` architecture while doing the exercises.
- Do not copy full implementations from examples.
- Prefer writing small tests and notes after each step.
- If stuck, ask for hints, not complete code.

## Spoiler Policy

This roadmap gives goals, constraints, hints, and reflection questions. It deliberately does not include full implementation snippets.

Good questions to ask while working:

- "What source file should I inspect next?"
- "What concept am I missing?"
- "Can you give me a smaller hint?"
- "Can you review my attempt?"

Questions to avoid during the challenge:

- "Write the whole implementation."
- "Give me the final code."
- "Solve the exercise for me."

## Suggested Location

Create the experiment inside the repo but away from the product path:

```text
experiments/
  pi-sandbox-lab/
```

This keeps the learning work visible without making it part of `apps/dev-cli`.

## Learning Outcomes

By the end, you should be able to answer:

- How comfortable do you feel with basic TypeScript project structure?
- What does embedded Pi feel like compared with CLI or RPC?
- How do Pi tools work at a practical level?
- How much can Pi tool operations be redirected?
- Where do `read`, `ls`, `find`, `grep`, `edit`, and `bash` differ in integration difficulty?
- How hard is Gondolin lifecycle management?
- What does sandboxed execution make easier or harder?
- Does RPC solve a real problem for `pi-mirror`, or is SDK embedding enough?
- What should change in `pi-mirror` after hands-on experience?

## Milestone 0: Lab Bootstrap

Goal:

- Create a tiny TypeScript project for experiments.

You are done when:

- `pnpm install` works.
- `pnpm test` runs at least one trivial Vitest test.
- `pnpm dev` or an equivalent script can run a tiny TypeScript entrypoint.
- The lab has a short `README.md` explaining that it is disposable.

Constraints:

- Use Node 24.
- Use pnpm.
- Use TypeScript.
- Use Vitest.
- Do not add Pi yet.

Hints:

- Start with the smallest package manifest that can run TypeScript.
- Make the first test embarrassingly small.
- Learn what `tsconfig.json` does before changing many options.
- Keep the lab package private.

Reflection prompts:

- What parts of TypeScript feel confusing?
- Do imports, exports, async functions, and test files make sense yet?
- What tooling do you want to understand before adding Pi?

## Milestone 1: TypeScript Basics Through Tiny Utilities

Goal:

- Get comfortable with TypeScript by writing small utilities that resemble future `pi-mirror` concepts.

Suggested exercises:

- Define a `WorkspaceId` type alias.
- Define a small `Workspace` object type.
- Write a function that resolves a relative path against a lab root.
- Write a function that rejects paths escaping the lab root.
- Test the path behavior with Vitest.

You are done when:

- You can explain the difference between a type, an interface, and a runtime value.
- You can write and run a few tests without help.
- You understand what is checked by TypeScript and what still needs runtime validation.

Constraints:

- Do not add Zod yet.
- Do not build the real `pi-mirror` domain model.
- Keep everything toy-sized.

Hints:

- TypeScript does not exist at runtime.
- Path safety is a runtime concern, not just a type concern.
- Tests are a good way to learn the edge cases without overdesigning.

Reflection prompts:

- Which bugs did TypeScript catch?
- Which bugs required tests?
- What did you expect TypeScript to do that it did not do?

## Milestone 2: Hello Pi Embedded

Goal:

- Start one Pi session from TypeScript and print a response.

You are done when:

- A lab script creates a Pi session.
- A simple prompt gets a response.
- You understand where model/auth/settings configuration lives for the experiment.

Constraints:

- No custom tools.
- No filesystem access.
- No sandbox.
- No RPC yet.

Hints:

- Prefer SDK embedding over shelling out to the Pi CLI.
- Look for the smallest first-party example of `createAgentSession`.
- Keep session storage local to the lab.
- Print enough lifecycle information to understand what is happening.

Reflection prompts:

- What does Pi own in this setup?
- What does the host script own?
- What setup felt magical or unclear?

## Milestone 3: One Boring Custom Tool

Goal:

- Learn the Pi tool-call flow with a harmless custom tool.

Suggested tool ideas:

- `get_lab_time`
- `echo_json`
- `read_lab_note`

You are done when:

- The agent can call your custom tool.
- The tool validates or handles simple input.
- The tool returns a result the model can use.
- Tool failures are visible and understandable.

Constraints:

- No file mutation.
- No shell execution.
- No sandbox.

Hints:

- Choose a tool so boring that the only hard part is Pi integration.
- Log tool inputs and outputs during the experiment.
- Make one intentional tool error and observe what Pi does.

Reflection prompts:

- What does a tool definition need?
- Where do argument schemas live?
- How does the agent decide to call the tool?
- What does a bad tool result look like?

## Milestone 4: Toy Workspace Read Tools

Goal:

- Give the agent read-only access to a fake workspace.

Suggested toy workspace:

```text
fixtures/
  toy-workspace/
    README.md
    notes/
    src/
```

Suggested tools:

- `list_files`
- `read_file`
- `search_files`

You are done when:

- The agent can inspect the toy workspace.
- It cannot read outside the toy workspace.
- The tools are tested without involving the model.

Constraints:

- Read-only.
- No `bash`.
- No Gondolin.
- No personal files.

Hints:

- Implement and test the underlying functions before exposing them as Pi tools.
- Treat path traversal as part of the exercise.
- Keep the tool names intentionally simple.

Reflection prompts:

- What should be enforced by the tool implementation?
- What should be explained to the model?
- What should be tested without an LLM involved?

## Milestone 5: Toy Workspace Write Tool

Goal:

- Add controlled writing to a scratch area inside the toy workspace.

Suggested rule:

- reads are allowed throughout `fixtures/toy-workspace`
- writes are allowed only under `fixtures/toy-workspace/scratch`

You are done when:

- The agent can create or edit a file in `scratch`.
- Attempts to write outside `scratch` fail.
- Tests cover allowed and denied writes.

Constraints:

- No shell execution.
- No sandbox yet.
- Do not implement profiles or config files.
- Hardcode the rule for learning.

Hints:

- Start with one write operation.
- Make denial messages clear.
- Do not rely on prompting as the enforcement mechanism.

Reflection prompts:

- What did hard guardrails feel like in practice?
- Where would this become annoying without a config model?
- How much policy can live in simple code before it needs structure?

## Milestone 6: Gondolin Without Pi

Goal:

- Learn Gondolin lifecycle separately from Pi.

You are done when:

- A script starts a Gondolin sandbox.
- It runs a simple command inside the sandbox.
- It can see a mounted toy workspace.
- It tears down cleanly.

Constraints:

- No Pi.
- No LLM.
- No personal files.
- Keep the command boring, such as listing files or printing the working directory.

Hints:

- Learn lifecycle before integrating tools.
- Print the sandbox-visible paths.
- Check what happens when a command fails.
- Check whether common binaries you expect are available.

Reflection prompts:

- How much setup does Gondolin require?
- How fast does it feel?
- What filesystem model does it expose?
- What failure modes are confusing?

## Milestone 7: Pi Read Tools Routed Into Gondolin

Goal:

- Make Pi read the toy workspace through the sandbox rather than directly from the host.

You are done when:

- The agent can list and read files that are visible inside Gondolin.
- Host paths are not exposed as the main mental model.
- Tool failures remain understandable.

Constraints:

- Read-only first.
- No shell tool exposed to the agent.
- Do not add long-term abstractions yet.

Hints:

- Keep a tiny adapter between Pi tools and the sandbox.
- Compare direct-host read tools with sandbox-backed read tools.
- Pay attention to path translation.

Reflection prompts:

- What became simpler with the sandbox?
- What became harder?
- Did the agent need different instructions?

## Milestone 8: The Grep And Find Reality Check

Goal:

- Understand which Pi/file tools are easy to redirect and which ones are awkward.

You are done when:

- You have tried a search operation inside the sandbox.
- You know whether search should be a custom tool, an operations override, or something else in future.
- You have notes on `grep` and `find` behavior.

Constraints:

- Do not solve this with a broad shell tool.
- Keep the experiment focused on search behavior.

Hints:

- Inspect Pi's tool source near the operation hooks.
- Compare how `find` and `grep` are implemented.
- Ask: where does the actual file traversal happen?

Reflection prompts:

- Which tool was easiest to sandbox?
- Which tool leaked local assumptions?
- What would this imply for `pi-mirror` tool ownership?

## Milestone 9: Controlled Shell In The Sandbox

Goal:

- Give the agent a tiny shell capability inside the sandbox and observe the risks.

You are done when:

- The agent can run a harmless command inside Gondolin.
- You can block or reject at least one risky command pattern.
- Command output and errors are visible.

Constraints:

- Toy workspace only.
- No host shell.
- No secrets.
- No package installs yet.

Hints:

- Start with explicit allowed commands before trying risk classification.
- Keep timeouts short.
- Observe how often the model reaches for shell when structured tools exist.

Reflection prompts:

- Does shell feel necessary?
- What did shell make easier?
- What did shell make scarier?
- What should `pi-mirror` default to?

## Milestone 10: RPC Contrast

Goal:

- Understand what Pi RPC is good for by building the smallest possible contrast with SDK embedding.

You are done when:

- You can start or connect to a Pi RPC flow.
- You can send a simple prompt and inspect the response path.
- You can explain when RPC might be useful.

Constraints:

- No product architecture decision yet.
- No custom UI.
- No sandbox integration unless it is trivial by this point.

Hints:

- Compare control flow, not features.
- Ask what process owns the session.
- Ask what becomes easier for non-Node hosts.

Reflection prompts:

- Is RPC simpler or more complex than SDK embedding for this use case?
- What would RPC buy `pi-mirror`?
- What would RPC make harder?

## Milestone 11: Mini Capstone

Goal:

- Ask the agent to perform a small task inside the sandboxed toy workspace.

Example task shape:

- inspect a toy project
- find a deliberately planted issue
- write a note or small patch in `scratch`
- explain what changed

You are done when:

- The agent completes the task without touching host files.
- You can inspect the resulting files.
- You can explain which parts were Pi, host tool code, and sandbox behavior.

Constraints:

- Keep the bug or task simple.
- Do not use personal files.
- Do not build promotion workflows.
- Do not polish the CLI.

Hints:

- Make the toy task small enough that failures teach you something.
- Save the transcript or notes from the run.
- Try the same task once with structured tools and once with shell available.

Reflection prompts:

- What surprised you?
- What did the agent do well?
- Where did your tool design make the agent worse?
- What should `pi-mirror` do differently because of this?

## Milestone 12: Retrospective And Architecture Update

Goal:

- Convert learning into better `pi-mirror` decisions.

You are done when:

- You have a short retrospective document.
- You have identified which ADRs still feel right.
- You have identified which ADRs or foundation sections need revision.
- You have decided whether the next product milestone should start.

Suggested retrospective sections:

- What I learned about TypeScript
- What I learned about Pi embedding
- What I learned about tools
- What I learned about Gondolin
- What I learned about RPC
- What I would change in `pi-mirror`
- What I still do not understand

Hints:

- Do not force every experiment into the product.
- Keep good ideas, but also keep the product smaller than the lab.
- Prefer updating ADRs only when a decision actually changes.

## Suggested Issue Breakdown

If turning this into GitHub issues, use one parent issue:

- `Learning spike: Pi Sandbox Lab`

Then create thin sub-issues:

- `Bootstrap TypeScript lab`
- `Learn Pi embedding with a no-tool session`
- `Add one harmless custom Pi tool`
- `Build read-only toy workspace tools`
- `Add controlled scratch writes`
- `Run Gondolin without Pi`
- `Route Pi read tools through Gondolin`
- `Investigate grep/find sandbox behavior`
- `Experiment with controlled sandbox shell`
- `Compare Pi RPC with SDK embedding`
- `Run mini capstone task`
- `Write retrospective and update architecture notes`

## Stop Conditions

Stop and revisit the plan if:

- the lab starts becoming a second product
- personal files are needed to continue
- the next step requires broad secrets or host permissions
- you feel tempted to design a full framework before finishing the experiments
- TypeScript confusion blocks Pi learning for more than one session

## Final Reminder

This is not a test of building the perfect agent.

It is a guided way to build enough intuition that the next `pi-mirror` decisions are based on experience instead of guesses.
