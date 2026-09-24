# 10 APIs and contracts

| Checkpoint | What counts as a problem |
| --- | --- |
| Zero-value semantics | The meaning of the zero value of a return value/parameter is not defined, or is defined but not documented |
| **Zero-value usability** | What happens when an uninitialized instance is used directly — if it works, say so; if not, block it; it must not be neither stated nor blocked |
| Construction completeness | Is the object usable right after construction; is there an initialization that must be called again; what happens if it is missed |
| Consistent handling of empty values | Among similar entry points, half return an error and half silently return a zero value |
| Error vs. zero value | The same failure returns an error from one entry point and a zero value from another (if the difference is deliberate, document it; if not, it is a bug) |
| Return signatures | Do multiple return values express independent, clear states; are there contradictory combinations, status flags that cannot be explained, or error returns that cannot be reached |
| Error and resource results | Does the combination of return value and error follow the project contract; are resources created before the failure released by this layer, is the close error kept, could the caller misuse a failed result |
| Parameter shape | Too many parameters, positional parameters whose meaning depends on remembering the order, boolean flag parameters |
| Ownership | Returned collections get modified internally; parameters that are taken over do not say "the caller must not write to it again" |
| **Borrowed parameters** | An object the caller lends to this layer (client, connection, config, buffer, callback) has its fields changed or its internal components swapped by this layer, or is kept and used after return; the contract does not say whether it is a read-only borrow or a takeover |
| **View lifetime** | Zero-copy slices, views, iterators and borrowed references become invalid after the next call, a reset, a return to a pool or a close; when they become invalid is not written down, and a caller that holds one too long reads someone else's data |
| **Callbacks versus the final result** | Callbacks and hooks are also called for probes, prechecks and candidate objects that are not used in the end, and may not be called at all on a cache hit; the state the caller records from a callback can differ from the object that finally takes effect, yet the contract does not say so and does not hand the result back together with the final object |
| Type leaks | Internal types appear in public signatures |
| Comparability | Are types used as map keys / in equality comparisons comparable; non-comparable members cause runtime crashes |
| Self-describing methods | Can to-string / to-error methods crash, call themselves recursively, or leak sensitive information |
| Interface implementation assertions | Is there a compile-time check that "this type really implements that interface" |
| Caller obligations | Call order, concurrency limits, must close, must not modify return values — are these written in the docs |
| Idempotency and retry | Is retrying after a failure safe; are repeated calls equivalent |
| Compatibility promises | The agreed compatibility range does not match reality, or compatibility burden that nobody asked for is left behind; for cleanup after a replacement see [1](1-dead-code-and-reachability.md) |

> Naming itself is checked in [6 Naming, comments and readability](6-naming-comments-and-readability.md); "is it easy to use wrong" is checked in [11 Usability and misuse resistance](11-usability-and-misuse-resistance.md).
