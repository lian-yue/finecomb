# 13 Error handling and failure semantics

| Checkpoint | What counts as a problem |
| --- | --- |
| Context | Errors are not wrapped, so the cause or location is lost |
| Swallowed errors | Among the explicitly discarded errors, some really need handling |
| Fallback branches | For fallback, catch-all, degraded, slow-path and "no match" branches taken when the main path does not apply: are the lengths, states and results they use computed on every path that reaches them, rather than left at their initial values; do empty, zero-length and single-element values, and the first item taken from a cache or queue, go through the same validation and setup as ordinary values |
| **Failures overwritten by later steps** | When several checks write the same result variable, error code or status bit, the success of a later check turns the failure of an earlier check back into success; a fallback or alternative verification that should only make up for one kind of failure (switching trust stores, switching verification methods, retrying) clears, once it succeeds, failures it is not responsible for |
| **Errors read as success** | One return value stands for success, failure and data at once (negative means error, zero means success or "none", a bit field mixes status and error codes), and the caller reads it by the wrong convention or handles only some of the failure values; failures of external dependencies, failed queries and timeouts are treated as "none" or "passed"; when an attacker causes a failure on purpose (running out of memory, overlong input, filling a queue, a concurrency conflict), which branch does the check fall into; is the decision made from the state actually reached (encrypted or not, the level granted, the version negotiated), not only from a success status |
| Error classes | "Expected states" and "real faults" are lumped together (target missing vs. I/O failure; end of stream vs. timeout) |
| Distinguishability | Callers cannot tell errors apart by type or sentinel value, only by string comparison |
| Retryability | Do errors separate "a retry might succeed" from "a retry is useless" |
| Partial success | The semantics of a batch operation that half fails are not defined; the meaning of the returned count is vague (does deleting one damaged fragment count as "deleted one record") |
| Multiple errors | When several things fail, is only the first reported or all of them; is this stated |
| Crashes | Deliberate crashes inside a library (panic, abort, throwing exceptions the caller should not have to handle); implicit crashes that public input can trigger reliably (out of bounds, null reference, failed type assertion or conversion, division by zero, comparing non-comparable or unhashable types) |
| Crash recovery | Catching hides an issue that should be fixed; **is the object still usable after the catch**; crashes or unhandled rejections in concurrent execution flows or async tasks that nobody handles take down the whole process or are silently dropped |
| Process-level handling | A library terminates the process directly (each language's exit, abort, uncaught fatal errors) |
| Boundaries of the exception mechanism | The catch scope is too wide and swallows interrupt, cancellation or exit signals; exceptions cross boundaries that do not support them (callbacks, cross-language calls, destructors); error return values are ignored. For language details see [Appendix A](../languages.md) |
| Actionability for people | The error says "what happened"; does it also say "what to do"; is the wording consistent |
| Sensitive information | Error messages carry paths, credentials, user data or internal structure — see [30](30-privacy-data-governance-and-compliance.md) |
