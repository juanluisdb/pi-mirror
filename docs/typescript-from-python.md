# TypeScript From Python: First Steps For Pi Sandbox Lab

This guide is for a Python developer learning just enough JavaScript and TypeScript to work on the Pi Sandbox Lab.

It assumes you already understand software engineering, tests, modules, functions, data structures, and async concepts. The goal is not to teach programming from scratch. The goal is to map what you know from Python onto the JavaScript/TypeScript ecosystem.

## Big Mental Model

JavaScript is the runtime language.

TypeScript is JavaScript plus a static type checker.

That means:

- TypeScript types mostly disappear at runtime.
- Runtime validation is still your job.
- A TypeScript program eventually runs as JavaScript on Node.
- `tsc` checks types.
- Node executes JavaScript.
- Tools like `tsx` let you run TypeScript directly during development.

Python comparison:

- JavaScript is closer to Python at runtime.
- TypeScript types are closer to Python type hints plus a stricter checker.
- TypeScript is usually enforced more heavily than Python typing in day-to-day projects.

One useful way to think about it:

- Python code usually runs first and is checked second, if at all.
- TypeScript is often checked before it runs.
- That means a lot of your early confusion will come from the type checker, not from the language runtime itself.

## A Practical First Setup

For the Pi Sandbox Lab, you want the smallest setup that teaches the right lessons without creating tool friction.

Recommended learning stack:

- Node 24
- `pnpm`
- TypeScript
- `tsx` for running TypeScript directly during development
- Vitest for tests

Very small project shape:

```text
experiments/pi-sandbox-lab/
  package.json
  tsconfig.json
  src/
    index.ts
    hello.ts
  test/
    hello.test.ts
```

Recommended scripts:

- `dev` for running a TypeScript entrypoint
- `test` for Vitest
- `typecheck` for `tsc --noEmit`

What to do first:

- create one pure function
- write one test for it
- run the test
- intentionally break it once so you can read the error shape

What not to do first:

- set up a big framework
- introduce Zod, path aliases, build pipelines, or bundlers before you need them
- worry about publishing or packaging

Helpful references for this setup:

- TypeScript handbook: https://www.typescriptlang.org/docs/
- Node API docs: https://nodejs.org/api/
- pnpm docs: https://pnpm.io/
- Vitest docs: https://vitest.dev/guide/

If a tool or configuration detail is confusing, read the docs for the smallest relevant concept first instead of jumping to package-level tutorials.

## The Runtime: Node

Node is the JavaScript runtime for backend and CLI programs.

For this project:

- use Node 24
- write CLI/lab scripts that run in Node
- do not worry about browsers

Python comparison:

- Node is like the Python interpreter plus standard runtime.
- `node script.js` is like `python script.py`.
- `process.env` is like `os.environ`.
- Node's `fs` module is like Python's file APIs.
- Node's `path` module is like `pathlib` / `os.path`.

Practical note:

- Node is not the browser.
- For this project, think "CLI and backend runtime" rather than "frontend JavaScript".
- If an example talks about the DOM or window objects, ignore it for now.

## The Package Manager: pnpm

`pnpm` installs dependencies and runs package scripts.

Python comparison:

- `package.json` is roughly like `pyproject.toml`
- `pnpm install` is roughly like `pip install -r ...` or `uv sync`
- `node_modules/` is the local dependency directory
- `pnpm-lock.yaml` is the lockfile
- `scripts` in `package.json` are like project task commands

Common commands:

```bash
pnpm install
pnpm test
pnpm dev
pnpm typecheck
```

Useful mental model:

- `package.json` describes what the project is and how to run it.
- `pnpm-lock.yaml` pins the exact dependency graph.
- `node_modules/` is the installed dependency tree.
- `pnpm` is the thing that coordinates all three.

If commands feel mysterious, inspect the scripts first.

For example:

- `pnpm test` usually means "run the test runner with repo-specific defaults"
- `pnpm dev` usually means "run the smallest useful entrypoint loop"
- `pnpm typecheck` usually means "ask TypeScript to validate without emitting files"

## The First Files

A tiny TypeScript lab usually has:

```text
experiments/pi-sandbox-lab/
  package.json
  tsconfig.json
  src/
    index.ts
  test/
    something.test.ts
```

Do not start with a complex framework.

Start with:

- one entrypoint
- one test
- one or two tiny utility functions

For the first day, you mostly want to be able to answer:

- where does code live?
- where do tests live?
- how do I run the script?
- how do I run the type checker?
- how do I know whether a failure is from TypeScript, Node, or the test runner?

If you can answer those, you are already in good shape.

## JavaScript Syntax Basics

Variables:

```ts
const name = "pi-mirror";
let count = 0;
```

Use `const` by default. Use `let` only when the binding changes.

Python comparison:

- `const` does not make the object deeply immutable.
- It means the variable binding cannot be reassigned.

Functions:

```ts
function greet(name: string): string {
  return `hello ${name}`;
}
```

Arrow functions:

```ts
const greet = (name: string): string => {
  return `hello ${name}`;
};
```

Objects:

```ts
const workspace = {
  id: "staging",
  access: "rw",
};
```

Arrays:

```ts
const tools = ["read", "ls", "grep"];
```

Conditionals look familiar:

```ts
if (workspace.access === "rw") {
  // ...
} else {
  // ...
}
```

Important difference from Python:

- use `===`, not `==`
- braces are required by convention
- semicolons are usually optional, but many projects keep them

Common beginner habit:

- use `const` for almost everything until you have a reason not to
- prefer plain object literals and arrays before reaching for classes
- keep functions small and named clearly

## Imports And Exports

Export a function:

```ts
export function normalizeName(name: string): string {
  return name.trim().toLowerCase();
}
```

Import it:

```ts
import { normalizeName } from "./normalize-name";
```

Python comparison:

- `export` marks what another module can import.
- Relative imports usually start with `./` or `../`.
- TypeScript/Node module resolution has more project configuration than Python imports.

Practical note:

- most early confusion comes from module resolution, not from syntax
- if an import path looks weird, the problem is often the project config rather than the function itself
- start with relative imports until the project grows enough to justify aliases

## Node Modules And ESM

Modern Node projects often use ESM-style imports and exports.

What that means in practice:

- files are imported with `import`/`export`
- Node treats modules differently depending on project config
- TypeScript and Node do not always agree on resolution rules unless the config is set up intentionally

For the lab, do not try to become a module-system expert on day one.

Instead:

- let the starter project pick a simple module style
- keep the first imports relative
- if something about file extensions or import paths feels strange, treat it as a config question, not a logic question

## Types, Interfaces, And Runtime Values

Type alias:

```ts
type WorkspaceId = string;
```

Interface:

```ts
interface Workspace {
  id: WorkspaceId;
  path: string;
  access: "ro" | "rw";
}
```

Use the type:

```ts
function canWrite(workspace: Workspace): boolean {
  return workspace.access === "rw";
}
```

Key idea:

- `Workspace` does not exist at runtime.
- It helps TypeScript check your code before runtime.
- If you parse JSON from disk, TypeScript does not magically know it is a valid `Workspace`.

Python comparison:

- `interface Workspace` is conceptually like a typed `Protocol` / `TypedDict` shape.
- But TypeScript checks object shapes structurally.
- If an object has the right fields, it can satisfy the interface.

Practical note:

- use `type` for unions and small aliases
- use `interface` for object shapes that you expect to pass around a lot
- do not overthink the difference at the beginning

## Union Types

TypeScript makes simple finite choices very nice:

```ts
type WorkspaceAccess = "ro" | "rw";
type WorkspaceKind = "staging" | "reference" | "canonical";
```

This is useful for `pi-mirror` because many concepts should be small closed sets.

Python comparison:

- similar to `Literal["ro", "rw"]`
- often easier to use than enums for simple strings

This pattern shows up everywhere in agent code:

- allowed tool names
- workspace kinds
- access modes
- profile names

Small closed sets are your friend.

## Optional Fields

```ts
interface Profile {
  id: string;
  writableWorkspace?: string;
}
```

The `?` means the property may be missing.

Important:

- missing and `undefined` are common in JS/TS
- `null` is a separate value
- good projects are explicit about when they use `null`

Python comparison:

- optional field is not the same as `Optional[str]` exactly
- it means the property may not exist

Related idea:

- `undefined` usually means "not provided"
- `null` usually means "intentionally empty"
- keep those meanings consistent within a project

## Async And Promises

JavaScript async functions return Promises:

```ts
async function loadConfig(): Promise<string> {
  return "config";
}
```

Call with `await`:

```ts
const config = await loadConfig();
```

Python comparison:

- `Promise<T>` is like an awaitable that eventually resolves to `T`
- `async` / `await` feels similar, but Node APIs and error handling have their own conventions

Practical note:

- most Pi and filesystem work will feel async
- once you get used to it, `async`/`await` becomes the normal shape of code rather than a special case
- do not try to "hide" async too early with clever wrappers

Errors:

```ts
try {
  const config = await loadConfig();
} catch (error) {
  console.error(error);
}
```

Important:

- caught `error` is usually `unknown`
- do not assume it is always an `Error` object

That matters a lot in TS because it forces you to check what you actually received before using it.

## When You Start Pi

You do not need to learn all of Pi before writing your first script.

Start by recognizing the shape:

```ts
// Shape only, not exact API.
async function runLabSession() {
  const session = await createAgentSession({
    /* model, auth, storage, settings */
  });

  return session;
}
```

The key ideas are:

- the host sets up the session
- Pi owns the agent runtime
- the first useful interaction is usually just "start session, send message, inspect response"

Useful Pi references:

- Pi SDK docs: https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/docs/sdk.md
- Pi tool index: https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/src/core/tools/index.ts
- Pi tutorial: https://gist.githubusercontent.com/dabit3/e97dbfe71298b1df4d36542aceb5f158/raw/213be5313e8e74470eb40333944bc829e2f6df5f/pi_tutorial.md

When you actually get to Pi work, go back to the sandbox lab roadmap and start with Milestone 2.

## Runtime Validation

TypeScript does not validate unknown input at runtime.

This is the single biggest "TypeScript is not Python" idea for the Pi Sandbox Lab.

When data comes from:

- JSON files
- command line arguments
- environment variables
- filesystem reads
- tool inputs

you still need runtime checks.

Example problem:

```ts
const raw = JSON.parse(text);
```

`raw` can be anything.

Later, `pi-mirror` will likely use Zod for runtime validation:

```ts
// Do not start here in the first exercise.
```

For the first learning steps:

- write small manual checks
- understand why runtime validation exists
- add Zod only after the pain is visible

A good rule:

- types protect your own code
- runtime validation protects your boundaries

## Error Shapes

You will see at least three kinds of failures:

- TypeScript errors before the program runs
- Node/runtime errors while the program runs
- test failures when behavior is wrong

That distinction is worth learning early.

If a file does not compile:

- look at the type error first

If a script crashes:

- look at the runtime error and the stack trace

If the behavior is wrong but the code runs:

- write or improve a test

## Files And Paths

Prefer Node's promise-based filesystem APIs:

```ts
import { readFile } from "node:fs/promises";

const text = await readFile("README.md", "utf8");
```

Use `node:path` for path handling:

```ts
import path from "node:path";

const fullPath = path.resolve(root, relativePath);
```

Python comparison:

- `node:fs/promises` is like async file utilities
- `node:path` is like `os.path`
- there is no direct built-in equivalent of Python's `pathlib` style unless you use libraries

## Testing With Vitest

Vitest is the test runner.

Basic shape:

```ts
import { describe, expect, it } from "vitest";

describe("canWrite", () => {
  it("allows rw workspaces", () => {
    expect(true).toBe(true);
  });
});
```

Python comparison:

- `describe` groups tests
- `it` is a test case
- `expect(...).toBe(...)` is an assertion
- similar role to `pytest`, different style

Practical note:

- keep tests tiny and focused on one behavior
- prefer one clear assertion over several vague ones
- when in doubt, test the smallest function you can

Start with tiny tests. They are here to teach the language and tooling.

## The First Exercises

Do these before adding Pi.

Suggested order:

1. Hello TypeScript
2. First Test
3. Workspace Shape
4. Safe Path Join
5. Async File Read

That order matters because it moves from pure functions to runtime checks to async I/O.

### Exercise 1: Hello TypeScript

Goal:

- run one TypeScript file

Challenge:

- create a function that returns a greeting
- call it from the entrypoint
- print the result

Hint:

- make the function pure first
- then call it from the script
- if you cannot run this, do not move on yet

### Exercise 2: First Test

Goal:

- prove Vitest works

Challenge:

- test the greeting function
- make one test fail intentionally
- read the error output
- fix it

Hint:

- do not test console output yet
- use this to learn how the test runner reports failures

### Exercise 3: Workspace Shape

Goal:

- learn object types

Challenge:

- define a tiny workspace type
- create two workspace objects
- write a function that returns whether a workspace is writable
- test it

Hint:

- use `"ro" | "rw"` instead of a boolean
- remember that a closed set of strings is often clearer than a boolean flag

### Exercise 4: Safe Path Join

Goal:

- learn runtime checks

Challenge:

- write a function that joins a root path and a relative path
- reject attempts to escape the root
- test normal and malicious-looking paths

Hint:

- do not trust string prefix checks until you understand path normalization
- use `path.resolve`
- compare the resolved path against the root after normalization

### Exercise 5: Async File Read

Goal:

- learn `async` / `await`

Challenge:

- read a file from a toy fixture directory
- return its text
- test the function

Hint:

- keep the fixture file tiny
- let the test create temporary files if that feels cleaner
- if file paths or permissions confuse you, stop and inspect the smallest possible read case first

## Common Python-To-TypeScript Traps

- Thinking types validate JSON at runtime. They do not.
- Using `any` too quickly. Prefer `unknown` until validated.
- Forgetting that object equality is by reference.
- Using `==` instead of `===`.
- Mixing `null`, `undefined`, and missing fields casually.
- Assuming imports work like Python packages.
- Putting too much code in the entrypoint instead of small exported functions.
- Skipping tests because "the compiler checked it."
- Expecting `dict` and plain object semantics to match exactly.
- Reaching for classes before you need them.
- Letting module config confusion block all progress.

## Helpful Habits

- Start with `const` and pure functions.
- Use small files.
- Make tests describe behavior, not implementation details.
- Prefer `unknown` at the edges and narrow it carefully.
- Keep `any` as a temporary escape hatch, not a default.
- Log or print the shape of values when you are debugging.
- Separate "what the type system knows" from "what actually exists at runtime."

## What To Learn Before Pi

Before adding Pi to the lab, you should be comfortable with:

- running a TypeScript entrypoint
- exporting and importing functions
- writing basic interfaces and union types
- using `async` / `await`
- reading files with `node:fs/promises`
- writing Vitest tests
- understanding the difference between type checking and runtime validation

You do not need to master:

- advanced generics
- decorators
- complex build tooling
- frontend TypeScript
- React
- publishing packages
- monorepo optimization

If you want a practical checkpoint before Pi, you should be able to:

- make a small function and export it
- import that function in another file
- run the code with Node or `tsx`
- write a test around it
- explain why a value passed TypeScript checks but still failed at runtime

## How To Ask For Help During The Challenge

Good requests:

- "Review this TypeScript and explain what is unidiomatic."
- "Give me a hint for why this type is failing."
- "Explain this compiler error using Python analogies."
- "Suggest the next smallest test."

Avoid:

- "Write the full lab bootstrap."
- "Implement the safe path function for me."
- "Give me the final Pi tool code."

## Success Criteria

You are ready to start the Pi milestones when:

- you can write a small function with typed inputs and outputs
- you can test it with Vitest
- you can read a file asynchronously
- you can explain why runtime validation still matters
- you can debug a basic TypeScript compiler error without panic

The goal is confidence, not mastery.
