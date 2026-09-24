# A.3 JavaScript and TypeScript

| Checkpoint | What counts as a problem |
| --- | --- |
| Loose equality and coercion | Type coercion in `==`; `typeof null === 'object'`; `NaN !== NaN` |
| **Numeric precision** | Every `number` is a double, so integers above 2^53 lose precision (especially common for 64-bit IDs in JSON); `BigInt` cannot be mixed with `number` in arithmetic, and `JSON` does not support it directly |
| **Prototype pollution** | Deep merge, assignment by path and similar code that handles external objects writes to `__proto__` or `constructor.prototype`; store external keys in `Object.create(null)` or a `Map` |
| Object key order | Integer-like keys come first in ascending numeric order; the other keys follow in insertion order |
| `this` and scope | `this` is lost in callbacks; `var` is function-scoped, so closures in a loop share one variable (`let` creates a new one per iteration) |
| **Floating promises** | A Promise without `await` or `.catch` loses its error or becomes an unhandled rejection (which terminates the process by default from Node 15); `async` callbacks in `forEach` are not awaited |
| await interleaving | Checking before an `await` and writing based on that check after the `await`, while another task has already changed the state in between (see [18](dimensions.md#18-concurrency-and-memory-model)) |
| `Promise.all` | Rejects as soon as one promise fails, while the rest keep running and their results are thrown away; use `allSettled` when all results are needed |
| Blocking the event loop | Synchronous CPU-heavy computation, synchronous file or crypto calls, and large `JSON.parse` calls block all requests |
| Streams and backpressure | Ignoring the `false` returned by `write()`; `pipe` does not propagate errors, use `pipeline` |
| Event listener leaks | Adding listeners again and again without removing them |
| Regular expressions | Catastrophic backtracking in backtracking engines; leftover `lastIndex` when a regex with the `g` or `y` flag is reused |
| Strings | `length` and indexing count UTF-16 code units, so truncation may split a surrogate pair |
| Array sorting | `sort()` compares as strings by default (`[10, 9, 1]` sorts to `[1, 10, 9]`) |
| Dates | Months start at 0; date-only ISO strings are parsed as UTC, while strings with a time but no time zone are parsed as local time |
| TypeScript types | Types exist only at compile time; asserting external input with `as` does no validation at all; `any` spreads; the non-null assertion `!`; `strict` is turned off |
| Modules | When both the CommonJS and the ES module copy are loaded, state splits in two; circular dependencies get uninitialized exports; side effects at module top level |
| Node processes and files | `child_process.exec` goes through a shell, while `execFile` and `spawn` do not by default; `path.join` does not stop path traversal, so normalize first and then check the prefix; `Buffer.allocUnsafe` returns uninitialized memory |
| Browser | `innerHTML`, `eval`, `new Function`, `setTimeout` with a string; `postMessage` without checking the origin (see [4.18](specialties.md#418-user-interfaces-and-accessibility)) |
| Installing dependencies | Install scripts (`preinstall`, `postinstall`) run arbitrary code; the lock file is not committed; private scopes are not bound to the private registry (dependency confusion) |
| Timers | `setInterval` callbacks overlap when they contain async work |
| Random numbers | `Math.random` is not a cryptographic random source; use `crypto.getRandomValues`, or Node's `crypto.randomBytes` and `crypto.randomUUID` |
