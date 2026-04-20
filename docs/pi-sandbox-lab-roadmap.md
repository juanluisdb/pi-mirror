# Pi Sandbox Lab Roadmap

This is a learning challenge before committing more implementation to `pi-mirror`.

The goal is to learn enough about Pi, TypeScript, RPC, tools, and Gondolin to make better architecture decisions later. This is intentionally not the product implementation.

## How To Use This Roadmap

The easiest way to use this is to move from top to bottom, but stop early if one milestone already taught you something important.

Recommended approach:

- Start with Milestone 0 and 1 to get comfortable with TypeScript and the lab setup.
- Do Milestone 2 before adding any tools.
- Only touch sandboxing and Gondolin after you have Pi sessions and a custom tool working.
- Keep short notes after each milestone about what felt easy, what felt surprising, and what still feels fuzzy.
- If you get stuck, ask for a hint on the smallest blocking step instead of skipping ahead.

What you are not trying to do yet:

- design the final `pi-mirror` product
- optimize for production-grade architecture
- solve every backend choice up front
- build a polished CLI or UX

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

## Reference Map

When you want to look up the real materials, start here:

- Pi SDK docs: https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/docs/sdk.md
- Pi tool index: https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/src/core/tools/index.ts
- Pi tutorial: https://gist.githubusercontent.com/dabit3/e97dbfe71298b1df4d36542aceb5f158/raw/213be5313e8e74470eb40333944bc829e2f6df5f/pi_tutorial.md
- Gondolin repo: https://github.com/earendil-works/gondolin
- TypeScript handbook: https://www.typescriptlang.org/docs/
- Node API docs: https://nodejs.org/api/
- Vitest docs: https://vitest.dev/guide/

How to use these links:

- read the overview first
- then inspect the smallest source or example that matches the concept you are learning
- prefer examples that show the shape of a thing, not just its marketing description

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

What this milestone is really teaching you:

- how a Node + pnpm + TypeScript project is put together
- how to run tests and a dev script
- where the first files usually live
- how much configuration you need before Pi enters the picture

Initial steps:

- create the experiment directory and make it separate from `apps/dev-cli`
- initialize a small `package.json`
- add a TypeScript entrypoint under `src/`
- add one test file under `test/`
- add scripts for `dev`, `test`, and `typecheck`
- keep the implementation tiny enough that you can read every file in one sitting

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
- If module or import errors appear, treat them as tooling lessons, not as signs that you are bad at the language.

Reflection prompts:

- What parts of TypeScript feel confusing?
- Do imports, exports, async functions, and test files make sense yet?
- What tooling do you want to understand before adding Pi?

## Milestone 1: TypeScript Basics Through Tiny Utilities

Goal:

- Get comfortable with TypeScript by writing small utilities that resemble future `pi-mirror` concepts.

What this milestone is really teaching you:

- the difference between runtime values and compile-time types
- how object shapes work
- how to write functions that are small enough to test well
- how to use literal unions instead of overengineering enums
- how to spot where runtime validation is still needed

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
- Use this milestone to notice where the compiler is helpful and where it is silent.
- Prefer a small working function plus a test over a clever abstraction.

Reflection prompts:

- Which bugs did TypeScript catch?
- Which bugs required tests?
- What did you expect TypeScript to do that it did not do?

## Milestone 2: Hello Pi Embedded

Goal:

- Start one Pi session from TypeScript and print a response.

What this milestone is really teaching you:

- what Pi owns versus what the host script owns
- how session creation feels in practice
- which setup pieces are configuration, auth, or model selection
- how much ceremony Pi needs before a first useful response

Pi session shape, conceptually:

```ts
// Pseudocode only: the exact method names depend on the Pi SDK version.
async function main() {
  const session = await createAgentSession({
    /* model, auth, storage, settings */
  });

  const reply = await session.send("Say hello in one sentence.");
  console.log(reply);
}
```

The important part here is not the exact API name. The important part is learning the split between:

- host-owned setup
- Pi-owned session lifecycle
- the message or prompt that starts the conversation

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
- Start without tools so you can see the bare session lifecycle first.
- If you are unsure what a setting does, write down the name and move on rather than freezing on configuration.

Reflection prompts:

- What does Pi own in this setup?
- What does the host script own?
- What setup felt magical or unclear?

## Milestone 3: One Boring Custom Tool

Goal:

- Learn the Pi tool-call flow with a harmless custom tool.

What this milestone is really teaching you:

- how Pi asks for tools
- what a tool signature needs to look like
- how arguments are validated or rejected
- how the agent reacts to tool success and tool failure

Custom tool shape, conceptually:

```ts
// Pseudocode only: treat this as a shape, not a final API.
const tools = [
  {
    name: "get_lab_time",
    description: "Return the current ISO time.",
    inputSchema: {},
    execute: async () => {
      return { time: new Date().toISOString() };
    },
  },
];
```

The useful questions here are:

- Where do tool names live?
- Where does input validation happen?
- What does the tool return?
- What happens when the tool fails?

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
- Keep the tool output small and predictable.
- Aim for one success path and one failure path, not a full utility library.

Reflection prompts:

- What does a tool definition need?
- Where do argument schemas live?
- How does the agent decide to call the tool?
- What does a bad tool result look like?

## Milestone 4: Toy Workspace Read Tools

Goal:

- Give the agent read-only access to a fake workspace.

What this milestone is really teaching you:

- how path scoping works
- where to enforce safety
- what the model can see versus what the host controls
- how read-only access feels before writes are allowed

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
- Make it obvious in logs when the agent is inside versus outside the toy workspace.

Reflection prompts:

- What should be enforced by the tool implementation?
- What should be explained to the model?
- What should be tested without an LLM involved?

## Milestone 5: Toy Workspace Write Tool

Goal:

- Add controlled writing to a scratch area inside the toy workspace.

What this milestone is really teaching you:

- what a hard guardrail feels like in practice
- how denial should behave
- why writes need stronger policy than reads
- how quickly the system becomes less trustworthy if writes are too broad

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
- If the write rule feels too rigid, that is useful feedback, not a failure.

Reflection prompts:

- What did hard guardrails feel like in practice?
- Where would this become annoying without a config model?
- How much policy can live in simple code before it needs structure?

## Milestone 6: Gondolin Without Pi

Goal:

- Learn Gondolin lifecycle separately from Pi.

What this milestone is really teaching you:

- how a sandbox starts and stops
- what it means to mount a workspace into an isolated environment
- what the sandbox can see by default
- what gets easier or harder once execution is isolated

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
- Treat this as a sandbox operator exercise, not an agent exercise.

Reflection prompts:

- How much setup does Gondolin require?
- How fast does it feel?
- What filesystem model does it expose?
- What failure modes are confusing?

## Milestone 7: Pi Read Tools Routed Into Gondolin

Goal:

- Make Pi read the toy workspace through the sandbox rather than directly from the host.

What this milestone is really teaching you:

- how much adapter code Pi needs when the filesystem is no longer local
- whether Pi tool wiring stays simple or starts getting awkward
- what it means to keep host paths out of the model's mental picture

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
- If the adapter feels thin, that is a good sign.
- If it becomes sprawling, write down why.

Reflection prompts:

- What became simpler with the sandbox?
- What became harder?
- Did the agent need different instructions?

## Milestone 8: The Grep And Find Reality Check

Goal:

- Understand which Pi/file tools are easy to redirect and which ones are awkward.

What this milestone is really teaching you:

- which tools naturally belong in the host
- which tools leak implementation assumptions
- whether you want a structured search tool or a shell-based search path later

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
- Focus on the shape of the integration problem, not on making search perfect.

Reflection prompts:

- Which tool was easiest to sandbox?
- Which tool leaked local assumptions?
- What would this imply for `pi-mirror` tool ownership?

## Milestone 9: Controlled Shell In The Sandbox

Goal:

- Give the agent a tiny shell capability inside the sandbox and observe the risks.

What this milestone is really teaching you:

- what happens when structured tools are not enough
- what kind of commands the model reaches for
- how hard it is to separate "useful shell" from "dangerous shell"

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
- You are trying to learn the pressure points, not to design perfect permissions.

Reflection prompts:

- Does shell feel necessary?
- What did shell make easier?
- What did shell make scarier?
- What should `pi-mirror` default to?

## Milestone 10: RPC Contrast

Goal:

- Understand what Pi RPC is good for by building the smallest possible contrast with SDK embedding.

What this milestone is really teaching you:

- what changes when the host and session are in separate processes
- what becomes easier if another language or controller owns the Pi session
- whether RPC is a better fit for your future than embedded SDK use

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
- Keep your notes focused on tradeoffs, not on feature count.

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
