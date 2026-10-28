# Testing and Tooling — Revision Notes (Beginner)

> One-line mental model: lock dependencies, lint for correctness, test behaviour once, build reproducibly, and let CI run the same commands.

## Core recall

- **npm and package.json**: manifest + `package-lock.json`. `dependencies` ship; `devDependencies` build/test only. `^3.0.0` = any 3.x, `~3.0.0` = 3.0.x, `3.0.0` = exact. `npm ci` = lockfile-only install that fails on mismatch.
- **Scripts**: `npm run <name>` uses `node_modules/.bin`; `npm test` is a shortcut; args after `--` go to the tool. `vitest` watches, `vitest run` exits.
- **ESLint**: finds problems (`no-unused-vars`, `no-undef`, `eqeqeq`) via flat `eslint.config.js`; severity `off`/`warn`/`error`. Fix or scope overrides.
- **Prettier**: formatter only — quotes, semicolons, width, trailing commas. `--check` in CI, `--write` locally. `eslint-config-prettier` removes overlap.
- **Vitest and Jest**: `describe` / `it` / `expect`; `toBe` (Object.is), `toEqual` (deep), `toBeCloseTo` (floats). Vitest uses `vi.fn/spyOn/mock`; Jest uses `jest.*` and injects globals.
- **Unit tests**: arrange → act → assert; one behaviour per test; assert return values and edge cases. `expect(() => fn()).toThrow()` for throws.
- **DOM testing**: `jsdom` + `environment: "jsdom"`; minimal DOM, `element.click()`, assert `textContent`. No layout: `getBoundingClientRect()` is zeros.
- **Mocking**: inject dependencies; `vi.fn().mockResolvedValue(x)`, `vi.spyOn(obj, "method")`, `vi.mock("./module")`; restore spies.
- **Coverage**: statements, branches, functions, lines; needs `@vitest/coverage-v8`. Measures execution, not correctness.
- **Vite and bundlers**: native ESM + esbuild in dev, Rollup build to `dist/`; `base` must match the deploy path; `server.proxy` avoids local CORS.
- **Source maps**: map generated to original positions; `sourcemap: true` / `"hidden"` / `"inline"`; makes stack traces readable.
- **Debugging**: breakpoints, `debugger`, `console.table/time/assert/trace`; Node `--inspect-brk`, `--enable-source-maps`.
- **CI basics**: clean machine, `npm ci` → lint → `test:run` → build; cache npm; fail fast.

## Must-remember code

```javascript
// one unit test
import { expect, it } from "vitest";
import { total } from "./cart.js";

it("sums price × qty", () => {
  expect(total([{ price: 10, qty: 2 }])).toBe(20);
  expect(total([])).toBe(0);
});
```

```javascript
// mock an injected dependency
const notify = vi.fn().mockResolvedValue(undefined);
await register({ name: "Asha", email: "asha@example.com" }, notify);
expect(notify).toHaveBeenCalledWith("asha@example.com", "Welcome Asha");
```

```javascript
// async rejection, awaited
it("rejects on 404", async () => {
  await expect(fetchUser(999)).rejects.toThrow("Not found");
});
```

```javascript
// DOM test setup
document.body.innerHTML = `<button id="inc">+1</button><p id="count">0</p>`;
document.querySelector("#inc").click();
expect(document.querySelector("#count").textContent).toBe("1");
```

## Interview traps

- A test that calls `.then()` without returning the promise finishes early and can pass falsely.
- `toBe` on objects/arrays compares references and fails; use `toEqual` or `toStrictEqual`.
- `toEqual` ignores `undefined` properties; `toStrictEqual` does not.
- `vitest` watch mode hangs CI; use `vitest run` or rely on `CI=true`.
- `npm install` in CI can drift with semver; use `npm ci` and commit the lockfile.
- ESLint is not a formatter and Prettier is not a linter; do not use one for the other's job.
- Mocking the function under test proves nothing; mock its dependencies.
- 100% coverage does not mean correct assertions.

## 3 things to remember

1. **`npm ci` in CI, `npm install` locally.**
   ```bash
   npm ci && npm run lint && npm run test:run && npm run build
   ```

2. **Test behaviour with exact assertions.**
   ```javascript
   expect(total(cart)).toBeCloseTo(119.5, 2); // not toBeTruthy()
   ```

3. **Await every async assertion.**
   ```javascript
   await expect(fetchUser(999)).rejects.toThrow("Not found");
   ```
