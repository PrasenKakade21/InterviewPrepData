# Error Handling — Revision Notes (Beginner)

> One-line mental model: throw meaningful `Error` objects, catch only what you can handle, preserve the `cause`, and clean up in `finally`.

## Core recall

- **try / catch / finally**: `catch` handles a thrown value; `finally` always runs — on success, throw, and `return`. A `return` in `finally` overrides the result.
- **throw**: throws any value, but always throw `Error` instances so `name`, `message`, `stack`, and `instanceof` work.
- **Error types**: `SyntaxError`, `TypeError`, `ReferenceError`, `RangeError`, `URIError`, `AggregateError` (has `.errors`). All extend `Error`.
- **Custom errors**: `class ValidationError extends Error`, call `super(message)`, set `this.name`, add fields such as `field`.
- **Error cause**: `new Error("context", { cause: err })` keeps the original error; read `err.cause`.
- **Async errors**: a throw in an `async` function rejects its promise. `try/catch` around `await` catches it; forgetting `await` does not.
- **Promise rejection**: end chains with `.catch()`. An unhandled rejection fires `unhandledRejection` / crashes Node.
- **Global handlers**: browser `error` + `unhandledrejection`; Node `process.on("uncaughtException")` + `"unhandledRejection"`. Logging only.
- **Re-throwing**: catch only to recover, translate, or add context. Otherwise `throw err` or wrap with `cause`.
- **Validation vs exceptions**: expected bad input → return a result; broken invariant → throw.
- **Reading stack traces**: top frame is the throw site; find the first frame in your own code. `file:line:column`.

## Must-remember code

```javascript
try {
  const data = JSON.parse(raw);
  return data;
} catch (err) {
  throw new Error("Invalid config", { cause: err });
} finally {
  cleanup(); // always runs
}
```

```javascript
class ValidationError extends Error {
  constructor(field, message) {
    super(message);
    this.name = "ValidationError";
    this.field = field;
  }
}
```

```javascript
// async: await inside try, or the catch never sees it
try {
  const user = await getUser(id);
} catch (err) {
  console.error(err.message);
}
```

## Interview traps

- `finally` runs **before** the caller gets the return value, and its own `return` wins.
- Throwing a string gives the catcher a string: `err.message` is `undefined`.
- A subclass does not set `name` automatically — it stays `"Error"` unless you set it.
- `catch` does not see an un-awaited promise rejection.
- `.catch()` in the middle of a chain does not protect later `.then()` throws.
- Global handlers are for reporting, not recovery — state may be corrupt.
- `err instanceof Error` fails for values thrown as plain objects or strings.
- Re-throwing without `cause` loses the original stack.
- Validating with exceptions for every typo makes normal flow slow and noisy.
- `JSON.parse` throws `SyntaxError`; check input before parsing when failure is expected.

## 3 things to remember

1. **Throw `Error` objects.**
   ```javascript
   throw new Error("Insufficient funds");
   // not: throw "Insufficient funds";
   ```

2. **`finally` always runs.**
   ```javascript
   try { risky(); }
   catch (err) { log(err); }
   finally { cleanup(); }
   ```

3. **Await inside try.**
   ```javascript
   try { await saveUser(data); }
   catch (err) { showError(err.message); }
   ```

> **Errors are values: throw real `Error` objects, catch only what you can handle, keep the `cause`, and never hide a failure.**
