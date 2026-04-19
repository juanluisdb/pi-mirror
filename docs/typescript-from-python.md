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

## Runtime Validation

TypeScript does not validate unknown input at runtime.

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

Start with tiny tests. They are here to teach the language and tooling.

## The First Exercises

Do these before adding Pi.

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

## Common Python-To-TypeScript Traps

- Thinking types validate JSON at runtime. They do not.
- Using `any` too quickly. Prefer `unknown` until validated.
- Forgetting that object equality is by reference.
- Using `==` instead of `===`.
- Mixing `null`, `undefined`, and missing fields casually.
- Assuming imports work like Python packages.
- Putting too much code in the entrypoint instead of small exported functions.
- Skipping tests because "the compiler checked it."

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
