# Testing and Tooling

Testing and tooling is the loop that turns a folder of `.js` files into a project you can change without fear. The chain is **install → configure → test → bundle → debug → automate**: `npm` records dependencies and scripts, ESLint and Prettier keep code consistent, Vitest runs tests, Vite builds, and CI repeats everything on each push.

```text
Project
   ↓
npm install       package.json + lockfile
   ↓
npm run lint      ESLint + Prettier
   ↓
npm test          describe → it → expect
   ↓
npm run build     Vite → dist/
   ↓
debug             source maps → original line
   ↓
CI                same commands, every push
```

---

# 1. Project and Scripts

### npm and package.json

`package.json` is the manifest: identity, module type, scripts and dependencies. `dependencies` ship to production; `devDependencies` are only for building and testing. `npm install` writes `package-lock.json`, pinning the exact tree for every machine.

```json
{
  "name": "kakade-quiz",
  "type": "module",
  "devDependencies": {
    "eslint": "^9.0.0",
    "vite": "^7.0.0",
    "vitest": "^3.0.0"
  }
}
```

Output:

```text
$ npm install
added 142 packages, and audited 143 packages in 4s

package-lock.json created — versions are now reproducible
```

`"type": "module"` makes `.js` files ESM; otherwise Node treats them as CommonJS. `^3.0.0` accepts `3.x`, `~3.0.0` only `3.0.x`, `3.0.0` only that version. `npm ci` installs strictly from the lockfile and fails on mismatch — use it in CI.

### Scripts

Scripts are named commands run with `node_modules/.bin` on `PATH`, so local tools work without global installs. `npm test` is a shortcut for `npm run test`.

```json
{
  "scripts": {
    "dev": "vite",
    "preview": "vite preview",
    "test:run": "vitest run",
    "coverage": "vitest run --coverage",
    "format": "prettier --write ."
  }
}
```

Output:

```text
$ npm run
  dev        vite
  test:run   vitest run
  coverage   vitest run --coverage

$ npm run test:run -- --reporter=dot
(extra arguments after -- go to the tool)
```

`vitest` watches; `vitest run` runs once and exits — the version CI needs. With `CI=true`, Vitest also switches to run-once automatically.

---

# 2. Code Quality

### ESLint

ESLint finds **problems**: unused variables, accidental globals, `==`, unreachable code. Modern ESLint uses a flat array in `eslint.config.js`; rules are `"off"`, `"warn"` or `"error"`.

```javascript
// eslint.config.js
export default [
  {
    files: ["**/*.js"],
    languageOptions: {
      ecmaVersion: 2024,
      sourceType: "module",
      globals: { window: "readonly", document: "readonly" },
    },
    rules: { "no-unused-vars": "error", "no-undef": "error", eqeqeq: "error" },
  },
];
```

Output:

```text
$ npx eslint .
src/cart.js
  4:7  error  'total' is assigned a value but never used  no-unused-vars
  9:5  error  Expected '===' and instead saw '=='          eqeqeq

✖ 2 problems (2 errors, 0 warnings)
```

Bad: disabling a rule globally because one legacy file fails. Good: fix it, or scope an override with `files: ["legacy/**"]`.

### Prettier

Prettier is an opinionated **formatter**. It ignores logic and rewrites whitespace, quotes, semicolons and line breaks so diffs stay small.

```json
{ "semi": true, "singleQuote": false, "printWidth": 80, "trailingComma": "all" }
```

Output:

```text
$ npx prettier --write src/cart.js
src/cart.js 18ms

Before: const cart={items:["pen","book"],total:0}
After:  const cart = { items: ["pen", "book"], total: 0 };
```

ESLint and Prettier overlap on formatting, so teams add `eslint-config-prettier` to switch off ESLint's stylistic rules. Lint for correctness, format for style.

---

# 3. Testing

### Vitest and Jest

Both use the same vocabulary: `describe` groups, `it` defines a case, `expect(value).matcher()` asserts. **Vitest** runs on the Vite pipeline, understands ESM natively and exposes `vi`; **Jest** is the older, widely deployed runner with `jest` globals.

| | Vitest | Jest |
| --- | --- | --- |
| Config | `test` block in `vite.config.js` | `jest.config.js` |
| ESM | native | needs transform |
| DOM | `environment: "jsdom"` | `testEnvironment: "jsdom"` |
| Mocks | `vi.fn()` | `jest.fn()` |

```javascript
// sum.test.js
import { describe, it, expect } from "vitest";
import { sum } from "./sum.js";

describe("sum", () => {
  it("adds two numbers", () => {
    expect(sum(2, 3)).toBe(5);
  });

  it("handles negatives", () => {
    expect(sum(-2, -3)).toBe(-5);
  });
});
```

Output:

```text
 ✓ sum.test.js (2 tests)
   ✓ sum > adds two numbers
   ✓ sum > handles negatives
```

Jest injects globals and uses `jest.config.js`; Vitest needs `globals: true`.

### Unit tests

A unit test checks one behaviour with no network, DOM or clock. Follow **arrange → act → assert**, and assert output instead of private internals. `toBe` uses `Object.is`; `toEqual` deep-compares; `toBeCloseTo` compares floats.

```javascript
// cart.js
export const total = (items) =>
  items.reduce((sum, item) => sum + item.price * item.qty, 0);

// cart.test.js
it("sums price × qty", () => {
  const cart = [
    { name: "pen", price: 10, qty: 2 },
    { name: "book", price: 99.5, qty: 1 },
  ];
  expect(total(cart)).toBeCloseTo(119.5, 2);
  expect(total([])).toBe(0);
});
```

Output:

```text
 ✓ cart.test.js (1 test)
   ✓ sums price × qty
```

Bad: `expect(cart.length).toBe(2)` — it tests the setup, not the function. Good: assert the return value plus the empty-array edge case.

### DOM testing

Install `jsdom` and set `environment: "jsdom"`. Build the smallest DOM in `beforeEach`, interact like a user, and assert rendered text.

```javascript
// greet.js
export function mountGreeting(root) {
  root.innerHTML = `<button id="greet">Greet</button><p id="out"></p>`;
  root.querySelector("#greet").addEventListener("click", () => {
    root.querySelector("#out").textContent = "Hello Kakade";
  });
}

// greet.test.js — beforeEach: reset body + mountGreeting(document.body)
it("greets on click", () => {
  document.querySelector("#greet").click();
  expect(document.querySelector("#out").textContent).toBe("Hello Kakade");
});
```

Output:

```text
 ✓ greet.test.js (1 test)
   ✓ greets on click
```

`jsdom` implements the DOM API, not layout: `getBoundingClientRect()` returns zeros and CSS is not applied. Test behaviour, not pixels.

### Mocking

A mock replaces a dependency you do not control — network, clock, email, storage. Prefer **injecting** it so the test is explicit; use `vi.spyOn` for existing objects.

```javascript
// register.js
export async function register(user, notify) {
  if (!user.email) throw new Error("Email required");
  await notify(user.email, `Welcome ${user.name}`);
  return { ok: true };
}

// register.test.js
it("notifies Asha once", async () => {
  const notify = vi.fn().mockResolvedValue(undefined);
  const result = await register({ name: "Asha", email: "asha@example.com" }, notify);

  expect(result).toEqual({ ok: true });
  expect(notify).toHaveBeenCalledTimes(1);
  expect(notify).toHaveBeenCalledWith("asha@example.com", "Welcome Asha");
});
```

Output:

```text
 ✓ register.test.js (1 test)
   ✓ notifies Asha once
```

For modules, `vi.mock("./email.js")` replaces the import for the whole file; `vi.spyOn(email, "send")` wraps one method. Restore spies, and never mock the function under test.

### Async tests

An async test must **return** or **await** its promise, or the test finishes before the assertion and passes falsely. `rejects` asserts a rejected promise.

```javascript
it("rejects on 404", async () => {
  await expect(fetchUser(999)).rejects.toThrow("Not found");
});

it("loads Kakade", async () => {
  expect((await fetchUser(1)).name).toBe("Kakade");
});
```

Output:

```text
 ✓ users.test.js (2 tests)
   ✓ rejects on 404
   ✓ loads Kakade
```

Bad: `fetchUser(1).then((u) => expect(u.name).toBe("Kakade"))` with no `return` — the test ends immediately. For timers use `vi.useFakeTimers()` and `vi.advanceTimersByTime(300)`, not real waits.

### Coverage

Coverage reports which lines, branches, functions and statements tests executed. It shows **what is untested**, not whether assertions are meaningful.

```bash
npx vitest run --coverage
```

Output:

```text
 % Stmts | % Branch | % Funcs | % Lines | Uncovered
---------|----------|---------|---------|----------
   92.3  |   81.8   |  100    |   92.3  | 24-26
```

Install `@vitest/coverage-v8` first; coverage is not bundled with Vitest. A threshold like `coverage: { thresholds: { lines: 80 } }` fails CI below 80%.

---

# 4. Build, Debug and Ship

### Vite and bundlers

Vite serves source as native ESM in development (esbuild transforms on demand) and produces a bundled, minified build with Rollup for production. The result is a static `dist/` folder.

```javascript
// vite.config.js
import { defineConfig } from "vite";

export default defineConfig({
  base: "/quiz/",
  build: { outDir: "dist", sourcemap: true },
  server: { proxy: { "/api": "http://localhost:3000" } },
});
```

Output:

```text
$ npm run build
✓ 24 modules transformed.
dist/index.html                 0.46 kB
dist/assets/index-a1b2c3.js    18.42 kB │ gzip: 6.10 kB
```

`base` must match the deployment sub-path or asset URLs 404. The dev proxy forwards `/api` to your backend, avoiding CORS locally. Hashed filenames let you cache assets forever.

### Source maps

A source map is JSON mapping generated positions back to original files. Browsers use it in DevTools; error trackers use it to make stack traces readable. `sourcemap: true` emits `.map` files; `"hidden"` emits them without a `sourceMappingURL` comment.

```json
{
  "version": 3,
  "file": "index-a1b2c3.js",
  "sources": ["../src/cart.js"],
  "sourcesContent": ["export const total = ..."],
  "mappings": "AAAA,SAAS,MAAM"
}
```

Output:

```text
Without map: TypeError at index-a1b2c3.js:1:4821
With map:    TypeError at src/cart.js:12:9
```

`inline` embeds the map in the bundle (bigger, self-contained). Public maps expose your source — usually fine for client code, but check before shipping proprietary logic.

### Debugging

Debugging is a loop: reproduce, inspect state, hypothesize, change one thing. `debugger` pauses when DevTools is open; breakpoints can be **conditional** (`i === 3`) or **logpoints** that print without stopping. `console.table` is handy for arrays of objects, `console.trace()` shows the call path.

```javascript
const total = [2, 1].reduce((sum, qty) => sum + qty, 0);
console.time("total");
console.assert(total === 3, "expected 3");
console.timeEnd("total");
console.log({ total });
debugger;
```

Output:

```text
total: 0.04ms        (varies by machine)
{ total: 3 }
(no assertion output — 3 === 3)
```

In Node, `node --inspect-brk app.js` opens the same DevTools; `--enable-source-maps` fixes traces.

### CI basics

Continuous integration runs the same checks on a clean machine for every push: install exactly, lint, test once, build.

```yaml
# .github/workflows/ci.yml
name: ci
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm }
      - run: npm ci
      - run: npm run lint
      - run: npm run test:run
      - run: npm run build
```

Output:

```text
ci / test (push)        ✓ 48s
  npm ci                ✓ 12s
  npm run lint          ✓ 6s
  npm run test:run      ✓ 9s
  npm run build         ✓ 14s
```

Notice: `npm ci` + lockfile pins reproducibility. Test **before** build so a red suite fails fast.

---

# Real-World Uses

- **Every PR** — lint, tests and build run in CI before review.
- **Bug fixes** — write a failing test first, then fix until green.

---

# Quick Revision

| Concept | Remember |
| --- | --- |
| **npm and package.json** | Manifest + lockfile; `npm ci` in CI |
| **Scripts** | `npm run <name>`; `npm test` shortcut; `--` forwards args |
| **ESLint** | Finds problems; flat config; correct, don't silence |
| **Prettier** | Formats only; `--check` in CI, `--write` locally |
| **Vitest and Jest** | `describe`/`it`/`expect`; `vi` vs `jest` |
| **Unit tests** | Arrange-act-assert; `toBeCloseTo` for floats |
| **DOM testing** | `jsdom`; click and assert rendered text |
| **Mocking** | `vi.fn()` / `vi.spyOn`; inject, then restore |
| **Coverage** | Lines/branches/functions; untested ≠ broken |
| **Vite and bundlers** | Native ESM in dev; Rollup build; hashed assets |
| **Source maps** | Minified → original file and line |
| **Debugging** | Breakpoints, `debugger`, `console.table` |
| **CI basics** | `npm ci` → lint → test:run → build |

## The Mental Model

```text
                 SOURCE
                   │
     ┌─────────────┼─────────────┐
     ↓             ↓             ↓
   lint         test           build
 ESLint       Vitest          Vite
 Prettier     jsdom/mocks     Rollup
     └─────────────┼─────────────┘
                   ↓
             dist/ + maps
                   ↓
            debug + monitor
                   ↓
                  CI
       same commands on every push
```

### The 3 things to remember

**1. Test behaviour, not implementation.**

```javascript
expect(total(cart)).toBeCloseTo(119.5, 2);
// not: expect(cart.reduce).toHaveBeenCalled()
```

**2. `npm ci` in CI, `npm install` locally.**

```bash
npm ci && npm run lint && npm run test:run && npm run build
```

**3. Source maps make production errors readable.**

```javascript
build: { sourcemap: true } // index-a1b2c3.js:1:4821 → src/cart.js:12:9
```

> **Tooling is a feedback loop: lock dependencies, lint for correctness, test behaviour once, build reproducibly, and let CI run the exact same commands on every push.**
