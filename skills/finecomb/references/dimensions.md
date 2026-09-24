# Part III: General dimensions

## 1 Dead code and reachability

| Checkpoint | How to check | What counts as a problem |
| --- | --- | --- |
| Unused declarations | Cross-check static analysis results with the call graph, registration mechanisms, build conditions and external APIs | Has no real use in any supported scenario; a public API cannot be judged dead only because nothing in the project references it |
| **Production code used only by tests** | First confirm what the tool can do, then confirm with production and test callers | Serves only tests and has no production duty; the difference between the tool's output in its two modes is only a lead |
| Unreachable branches | Follow config constraints and callers backward to find the real value domain of each branch | Branches that upstream validation already rules out (e.g. a parameter is validated as positive, but the code still keeps a whole path for it being ≤ 0) |
| Leftovers of old implementations | Look for aliases, wrappers, forwarders, old paths, deprecation markers, comments like "used to be called X" | A new implementation has replaced the old one, but the old one is still there |
| Speculative design | Look for parameters nobody passes, fields nobody reads, switches not wired up, interfaces with only one implementation and no need for substitution | "In case we need it later" |
| Dead tests | Unused helpers, cases that only cover deleted paths | See [41](#41-test-quality) |

**Decision rule**: classify only after confirming the real use, the reachability conditions and any explicit intent to keep it. Whether to delete it, keep it commented out or change platform code follows the project rules; a read-only review only reports the basis for the classification.

## 2 Leftover markers and suppressions

> These are not dead code. They are **unfinished items and silenced warnings that others left behind**. Each one stands for a decision or a debt, and the review must judge, one by one, whether each still holds.

| Checkpoint | How to check | What counts as a problem |
| --- | --- | --- |
| TODO / FIXME / XXX / HACK / to-verify | Full-text search | Judge each one: does it still hold? Whose responsibility is it? Should it be done, deleted or written into the docs? Do not assume "leaving it there is fine" |
| Commented-out code | Search for runs of commented-out code lines | Is the reason for stopping written down; has the commented-out content drifted away from the current implementation |
| **Static analysis suppressions** | Search for the suppression markers of lint, type checks and security scans in each language (for the syntax, see the "Leftover markers" row in [Part VII](tools.md)) | Behind each suppression is a decision or an unfixed issue; does the reason still hold, has it gone stale as the code changed |
| Empty implementations / no-ops | Look for implementations whose body is empty, only returns zero values, only does `pass` or only throws "not implemented" | Is it an intentional empty implementation or unfinished work; an intentional one must say so |
| "Unreachable" crash points | Search for explicit termination and assertions: panic, abort, assert, unreachable, fatal, process exit calls, "should not reach here" exceptions | Is it really unreachable? Can public input reach it? Are the assertions still there in release builds (see [16](#16-language-and-runtime-pitfalls))? |
| Debug leftovers | Hard-coded switches, always-true conditions, temporary prints (each language's print, console, debug log level), breakpoint statements, hard-coded test addresses and accounts | Slipped into production |
| **Leftover backups and build outputs** | Backup directories, copies and build outputs in the source tree | Should be cleaned up or have a clear reason to stay; they pollute search results and static analysis |
| "For now", "later", "TBD" in documents | Full-text search | Stale plans taken as the current contract |

## 3 Duplication

| Checkpoint | How to check |
| --- | --- |
| Line-by-line duplicate functions | Compare the implementations of the same concept (e.g. a background job and a foreground entry point each wrote an identical scan) |
| Copy-pasted inline logic | A piece of logic exists as its own function and is also copied by hand somewhere else (e.g. a function hand-copies the whole body of another function that already exists) |
| The same check scattered in many places | The same fail-fast check is written once in each of three entry points |
| Synonymous entry points | Two public entry points only forward to each other, and their docs use the same wording |
| Parallel types | Two structures express the same thing |
| Duplicate constants | The same magic number is written separately in many places |
| Cross-module duplication | Another module in the project already has an equivalent capability (see the comparison method in [Review method](scope.md#review-method)) |
| Documentation duplication | The same fact is written separately in several documents and comments; change one and the others drift — see [43](#43-documentation-consistency) |

## 4 Abstraction and layering

| Checkpoint | What counts as a problem |
| --- | --- |
| Wrapper layers | A layer that only renames and adds no behavior |
| Number of callers of an abstraction | An interface or abstraction with only one caller and no need for substitution, test doubles or dependency inversion |
| Interface width | Interface methods that nobody calls |
| Configuration vs. per-call policy | Long-lived configuration mixed with per-call policy; for concrete parameter misuse see [10](#10-apis-and-contracts) and [11](#11-usability-and-misuse-resistance) |
| Ownership of responsibility | Logic that serves only one object is split out into free-floating functions; the object's own state/lifecycle is not expressed by its own methods |
| Layer ownership | Business policy pushed down into a general-purpose layer; specific business, deployment or product policy shows up in a general-purpose layer |
| Layer responsibilities | When there are two layers (such as a handle layer / storage layer), does one layer overstep or re-implement the other's responsibility |
| File organization | Main-flow objects do not have their own files; vague catch-all files exist |

> Dependency direction, dependency cycles and layer inversion are checked in [5 Coupling and blast radius](#5-coupling-and-blast-radius); the size of functions and files is checked in [7 Complexity and maintainability](#7-complexity-and-maintainability).

## 5 Coupling and blast radius

> Most "fix one place, break another" issues are not inside a single function but in **the relationships between things**.

| Checkpoint | How to check | What counts as a problem |
| --- | --- | --- |
| Call graph | List the target's callers and callees; who in the project references it | Treated as "unused" when it actually has callers; the blast radius of a change is not worked out |
| **Multiple copies of the same fact** | List every place the same data lives: memory, disk, counters, summary structures, indexes, remote | No clear time or owner for bringing the copies back in line; the inconsistency window is not defined or documented |
| Linked invariants | Look for paired state where "if A changes, B must change too" | Paths exist that change only half — **especially error paths and early returns** |
| Implicit coupling | Dependencies built through global variables, file system layout, naming conventions or execution order | The dependency is not expressed in code; changing one place silently breaks another |
| Consistency of two-way mappings | Two ways of locating the same object (such as "derive an index from a name" and "build a path from a name") | The two algorithms disagree → written at A, looked up at B |
| Dependency direction | Does a lower layer know about an upper layer; are there cycles | Layer inversion, dependency cycles |
| Initialization order | Package/module-level initialization, global registries, singletons, default-value wiring | Order dependencies not written down; I/O or things that can fail done during initialization |
| Cross-module assumptions | The assumptions this target makes about other modules' behavior | The assumption is not guaranteed by the other module's public contract |
| Blast radius of a change | For each change, list: whose behavior changed, whose docs must change, whose tests must change | Callers, docs or the test matrix missed |

## 6 Naming, comments and readability

| Checkpoint | What counts as a problem |
| --- | --- |
| Concept consistency | The same concept has different names in different places; the same name means different things in different places |
| Misleading names | A getter has side effects; a predicate changes state; the name promises more or less than the implementation does |
| Name does not match reality | The name describes past behavior (e.g. a field refreshed on every write but named "creation time") |
| Negative names | Double-negative booleans (`notDisabled`, `ignoreNoCheck`) |
| Units in names | A timeout does not say seconds or milliseconds; a size does not say bytes or item count |
| Abbreviations and spelling | Invented abbreviations; misspellings frozen into a public API |
| Scope and name length | Short names in long scopes; long names for one-line locals |
| Comments explain "why" | Comments only restate what the code does, not why it does it this way or why not another way |
| Missing comments on key constraints | Non-obvious preconditions, side effects, concurrency requirements, performance traits or failure semantics are not written down |
| Stale or misleading comments | Comments describe an old implementation; comments mention names that do not exist |
| Comment rules | Check comment language, structure and opening wording against the project rules; **for multilingual comments, check that every language is complete** (a common mistake: only one language mentions some return value) |
| Readable at a glance | Can someone unfamiliar with the code answer "under what conditions does it get here, and what happens on failure" from the code and comments alone |

## 7 Complexity and maintainability

| Checkpoint | How to check | What counts as a problem |
| --- | --- | --- |
| Branch complexity | Complexity metric tools | A single function has so many branches that a person cannot read it in one pass or enumerate its tests |
| Nesting depth | Metrics or manual reading | Deeply nested conditions and loops; can early returns flatten them |
| Function length | Metrics | One function does too many things |
| File length and cohesion | Metrics + manual reading | One file carries several unrelated responsibilities |
| Number of exits | Manual reading | Too many scattered exits, making it hard to check that cleanup is complete on each exit |
| Cognitive load | Manual reading | How many contexts must be held in mind at once to understand a passage |
| Automated constraints | Check the checks and configuration the project already requires | Agreed checks are missing or broken; having no automated complexity gate is not a defect by itself |

> Parameter count, interface width and the number of abstraction layers are checked in [4 Abstraction and layering](#4-abstraction-and-layering) and [10 APIs and contracts](#10-apis-and-contracts).

## 8 Functional correctness and fitness for requirements

> The earlier dimensions ask "is the code well written". This one asks the most basic question: **does it do what it should, does it do all of it, and does it do it right.**

| Checkpoint | What counts as a problem |
| --- | --- |
| Requirement coverage | Is every claimed capability implemented; is anything "in the docs but not in the code" |
| Functional suitability | Is what was built actually needed; was anything built that nobody wants |
| Core correctness | Are the main-path results right; have they been compared with the authoritative definition, a reference implementation or known test vectors |
| **Source of the spec** | What is the behavior based on — a spec, a standard, upstream docs, or "it looks like it should be this way"; are the places with no basis marked |
| Following external standards | When implementing a standard / protocol / format, are all required parts done; are the choices on optional parts written down; **are deviations from the standard recorded with reasons** |
| Boundary semantics | Is the behavior on empty input, a single element and extreme values what the requirement wants (not just "does not crash") |
| Implicit premises | Do the idempotency, reentrancy, ordering and identity-comparison semantics that the calling scenario or upper-layer contract actually relies on hold; requirements that cannot be derived must not be made up |
| Paired capabilities | Are the create and delete, acquire and release pairs the contract requires complete; judge objects released by an external owner, or explicitly append-only, by their actual duties |
| Acceptance method | Does each claimed capability have a matching way to verify it; what proves it works |

## 9 Business logic and flow integrity

> This dimension looks for issues where **"each step is right on its own, but together they can be bypassed"**. Every technical check passes, yet the business rule is still broken through.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Step order** | Can a multi-step flow skip steps, run out of order or repeat; does the server enforce the order, or does it rely only on the UI not letting you click |
| State preconditions | Is each step's requirement on the prior state checked on the authoritative side |
| Timing and races | What happens when the same flow runs twice at the same time (double claiming, over-issuing, double spending) |
| Quantities and quotas | Negative, zero and huge quantities; atomicity of quota deduction and restoration — see [23](#23-transactions-atomicity-and-consistency) |
| Pricing and amounts | Is the amount passed in by the requester or computed on the authoritative side; stacked discounts, rounding, currency |
| Replay | What happens when the same request is replayed; is there a one-time token or an idempotency key |
| **Anti-automation** | Brute force, bulk enumeration, scalping (bots grabbing limited stock), scraping — is there rate limiting, a challenge, anomaly detection; by what dimension are limits applied (account / address / device / global) |
| Enumeration leaks | Can resources or accounts be enumerated through differences in responses (exists or not, fast or slow, error wording) |
| Single authority for rules | Are business rules written separately in many places, or is there one implementation; when two places disagree, which one wins |
| Reverse operations | Do the undo, refund and rollback paths have equally strong checks |
| Time windows | Can validity periods, cooldowns and deadlines be bypassed (changing the client time, sitting right on the boundary) |

## 10 APIs and contracts

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
| Type leaks | Internal types appear in public signatures |
| Comparability | Are types used as map keys / in equality comparisons comparable; non-comparable members cause runtime crashes |
| Self-describing methods | Can to-string / to-error methods crash, call themselves recursively, or leak sensitive information |
| Interface implementation assertions | Is there a compile-time check that "this type really implements that interface" |
| Caller obligations | Call order, concurrency limits, must close, must not modify return values — are these written in the docs |
| Idempotency and retry | Is retrying after a failure safe; are repeated calls equivalent |
| Compatibility promises | The agreed compatibility range does not match reality, or compatibility burden that nobody asked for is left behind; for cleanup after a replacement see [1](#1-dead-code-and-reachability) |

> Naming itself is checked in [6 Naming, comments and readability](#6-naming-comments-and-readability); "is it easy to use wrong" is checked in [11 Usability and misuse resistance](#11-usability-and-misuse-resistance).

## 11 Usability and misuse resistance

> A library's "usability" is not about how nice the interface looks. It is about **how hard it is for others to use it wrong**. A clearly written contract does not mean it is hard to misuse.

| Checkpoint | What counts as a problem |
| --- | --- |
| Least surprise | Is the behavior consistent with the name and with the conventions of similar interfaces |
| **Hard to misuse** | Are there usages that are "easy to get wrong yet still compile"; can types, scopes or required parameters stop wrong usage at compile time |
| Dangerous defaults | Can a caller trigger dangerous behavior beyond what they expect without knowing it; for concrete default values see [32](#32-configuration-and-defaults) |
| Confusable parameters | Adjacent parameters of the same type are easy to swap (two strings, two durations, two identifiers) |
| Calls that must be paired | Acquire/release, begin/end, lock/unlock — what happens if one half is missed; can a scope mechanism force the pairing |
| Partial initialization | Can a "half-built" object be constructed and used |
| Silent failure | Used wrong but with no feedback at all (returns a zero value, ignores a parameter, silently degrades) |
| Actionable error messages | After misuse, does the error point straight at how to fix it |
| Examples are the contract | Are the doc examples the recommended usage; does copying an example walk into a trap |
| Escape hatches | When a high-level wrapper blocks low-level capabilities, is there a way out; is there a risk the way out gets abused |

## 12 State machines and transitions

| Checkpoint | How to check | What counts as a problem |
| --- | --- | --- |
| State enumeration | List the states the object actually has and how each is represented | State combinations contradict each other, transition conditions are unclear; not using an enum type is not a defect by itself |
| **State × operation matrix** | Put the behavior of each public operation in each state into a table | Some cells are empty — not defined, not tested, not documented |
| Illegal transitions | Reopen after close, use before initialization, start twice, skip intermediate states | Illegal transitions silently accepted, or behavior undefined |
| Transition atomicity | Can others see an intermediate state during a transition | The intermediate state is observable and not defined |
| Terminal states | Can it recover after entering an error state; is the terminal state really terminal | Stuck in an intermediate state with no way out |
| Concurrent transitions | Two execution flows trigger a transition at the same time | See [18](#18-concurrency-and-memory-model) |
| State and resources | Which resources are held in each state; who releases them on a transition | Release missed on some transition path |
| State persistence | Is state kept across restarts; which state does it land in after a restart | The state after restart does not match reality |

## 13 Error handling and failure semantics

| Checkpoint | What counts as a problem |
| --- | --- |
| Context | Errors are not wrapped, so the cause or location is lost |
| Swallowed errors | Among the explicitly discarded errors, some really need handling |
| Error classes | "Expected states" and "real faults" are lumped together (target missing vs. I/O failure; end of stream vs. timeout) |
| Distinguishability | Callers cannot tell errors apart by type or sentinel value, only by string comparison |
| Retryability | Do errors separate "a retry might succeed" from "a retry is useless" |
| Partial success | The semantics of a batch operation that half fails are not defined; the meaning of the returned count is vague (does deleting one damaged fragment count as "deleted one record") |
| Multiple errors | When several things fail, is only the first reported or all of them; is this stated |
| Crashes | Deliberate crashes inside a library (panic, abort, throwing exceptions the caller should not have to handle); implicit crashes that public input can trigger reliably (out of bounds, null reference, failed type assertion or conversion, division by zero, comparing non-comparable or unhashable types) |
| Crash recovery | Catching hides an issue that should be fixed; **is the object still usable after the catch**; crashes or unhandled rejections in concurrent execution flows or async tasks that nobody handles take down the whole process or are silently dropped |
| Process-level handling | A library terminates the process directly (each language's exit, abort, uncaught fatal errors) |
| Boundaries of the exception mechanism | The catch scope is too wide and swallows interrupt, cancellation or exit signals; exceptions cross boundaries that do not support them (callbacks, cross-language calls, destructors); error return values are ignored. For language details see [Appendix A](languages.md) |
| Actionability for people | The error says "what happened"; does it also say "what to do"; is the wording consistent |
| Sensitive information | Error messages carry paths, credentials, user data or internal structure — see [30](#30-privacy-data-governance-and-compliance) |

## 14 Boundaries, numbers and text

| Checkpoint | What counts as a problem |
| --- | --- |
| Integer conversion | Truncation when converting a wide type to a narrow one; integers whose width depends on the platform |
| Overflow | Addition/multiplication overflows into a negative number; `limit + 1` overflows; capacity calculations overflow. Whether overflow wraps around, throws, saturates or is undefined behavior depends on the language and build mode; confirm which |
| Arbitrary-precision numbers | No overflow, but a cost: huge integers or very high-precision decimals built from external input exhaust CPU and memory in arithmetic, base conversion or serialization |
| Integers carried in doubles | In languages or formats that represent all numbers as double-precision floats, integers above 2^53 (such as 64-bit IDs) lose precision after parsing |
| Extreme values | Zero, negative numbers, maximum and minimum values are not handled or not tested |
| Arrays and slices | Out of bounds; length and capacity mixed up |
| Empty and missing | How empty input, null references and empty collections are handled |
| Floating point | Equality comparisons; accumulated precision error; **exact values such as money or measurements must not use floating point** |
| Non-finite values | Can NaN, positive and negative infinity and negative zero get into comparisons, sorting, quotas and serialization through parsing or computation; does a failed comparison unexpectedly bypass a value constraint |
| Rounding | Is the rounding rule (round half up / banker's rounding / truncation) clear and consistent; is the accumulated error acceptable |
| **Text and encoding** | Byte length / code points / grapheme clusters / display width mixed up; locale-dependent case conversion; equivalent text judged unequal because of different Unicode normalization forms; how invalid encoded bytes are handled; can truncation cut a character in half |
| Failure midway through parsing | Has external state already been changed when parsing fails; leftovers after a partial parse |
| Domain constraints | At which layer are business-level value constraints (non-negative, monotonically increasing, closed enum sets) checked; are there bypass paths |

## 15 Algorithm and data structure correctness

| Checkpoint | What counts as a problem |
| --- | --- |
| Comparison functions | The comparison used for sorting is not a strict weak ordering (returns true both ways for equal elements, not transitive) → results out of order or even a crash |
| Overflow in comparisons | Subtraction used as the comparison result; subtracting two large numbers overflows → order reversed |
| Sort stability | A stable sort is needed but an unstable one is used; is the relative order of equal elements promised |
| Rounding direction | Up or down; can small values get stuck at 0 (a classic trap in ratio and quota calculations) |
| Modulo bias | Bias from taking random numbers modulo; using a mask as modulo requires the capacity to be a power of 2 |
| Hash usage | When the same hash is used for sharding in two places and both take the same bits → strongly correlated, the distribution degrades |
| Shuffling | Wrong range in the shuffle algorithm → uneven distribution |
| Binary search and ranges | Open vs. closed ranges, updates to the upper and lower bounds, termination conditions |
| **Map iteration order** | Relying on an iteration order that the language or container does not promise (some are random, some follow insertion order, some sort part of the keys; different containers in the same language also differ; see [Appendix A](languages.md)) |
| **Serialization order stability** | Byte strings generated from unordered containers (config, query strings, checksum input) have an unstable order → unstable identity or checksum |
| Identity normalization | Are equivalent spellings of an input normalized to the same identity; can the normalization rule **wrongly merge inputs that are not equivalent** (such as lowercasing paths, see [27](#27-security-and-trust-boundaries)) |
| Set operations | Boundaries of dedup, intersection, union and difference; empty sets |

## 16 Language and runtime pitfalls

> This section lists **pitfall categories that apply across languages**. Each category must be applied to the languages the target actually uses: first find the concrete mechanism for that language in [Appendix A](languages.md); for a language not in the appendix, find the counterpart of each category yourself and write down the basis in the coverage record. For a mixed-language target, go through each language separately; for cross-language boundaries also see [4.32](specialties.md#432-cross-language-boundaries-and-native-extensions). Semantics can change with the language version and build mode, so first confirm the version the target declares and actually uses.

| Category | How to check | What counts as an issue |
| --- | --- | --- |
| **Null and missing values** | Read the places that use nullable values, optional values, interfaces, errors or map lookup results | Null has several representations (null pointer, an interface holding a null pointer, undefined vs. null, a missing key) but only one is checked, so the caller always sees an error or never sees one |
| Values, references and aliasing | Assignment, argument passing, slices and views, shallow copies, default arguments, loop variables | Thought to be copied but actually shared, and changed from far away; after truncating and reusing, old references still point to the same storage; mutable default values shared across calls; the loop variable is a copy, so changing it has no effect; unnecessary copies of large objects |
| **Copied synchronization objects** | Static analysis + manually checking when the object is used | Breaking a synchronization type's copy contract, e.g. copying a lock, atomic, wait group or condition variable after first use, so the protection is split off from the data that is really shared |
| **Alignment, width and atomicity** | Fields used in atomic operations, cross-platform structs, shared memory | The width or alignment an atomic operation needs is not met on some platforms (such as 64-bit atomics on 32-bit platforms); plain reads and writes treated as atomic; a "will not tear" assumption with no language guarantee |
| Integer semantics | Arithmetic, type conversion, shifts | Whether overflow wraps around, throws, saturates or is undefined behavior varies by language and build mode, but the check is written for the wrong one; signed and unsigned mixed |
| Undefined behavior and memory safety | Languages with undefined behavior or manual memory management, and each language's unsafe blocks | Out of bounds, use after free, double free, uninitialized reads, data races; the compiler relies on undefined behavior to remove checks that seem to be there |
| Dispatch, interface satisfaction and inheritance | Method sets, virtual functions, prototype chains, method resolution order, overloading and overriding | Thought to implement an interface or override a method but it does not (the receiver form or signature is slightly different); the base class destructor is not virtual; object slicing |
| **Embedding promotion and optional capabilities** | Embedding, inheritance, mixins; places that probe for capabilities (type assertions, `hasattr`, feature detection) | Methods of an embedded type accidentally promoted into public API; after a wrapper embeds a concrete type, a caller probing for an optional capability finds the promoted underlying method and **bypasses the wrapper's logic** (encryption, metering, rate limiting, auditing); or the wrapper drops the underlying optional capabilities (half-close, zero-copy, raw handle) |
| **Implementer duties of standard protocols** | Implementing interfaces, protocols and magic methods defined by the language or standard library | The documented rules of read/write, iteration, comparison, hashing, context management, close, stringification and similar protocols are not met. E.g.: a read can return data and an error at the same time; a short write must report an error; a timeout error must be recognizable as a timeout by the caller; equal objects must have equal hashes; whether close can be called more than once must follow the convention |
| Equality, hashing and comparison | Types used as keys, for dedup, sorting and comparison | Non-comparable or unhashable members crash at runtime; equality and hashing disagree; a mutable object is changed after being used as a key; NaN is not equal to itself |
| Implicit conversion and truthiness | Comparisons, conditions, mixed-type arithmetic | Loose equality, automatic string-to-number conversion, signed vs. unsigned comparisons, and differences in the truthiness of empty collections/zero/empty strings let validation be bypassed |
| Closure capture | Check the language version, where the variable is declared and when it is used asynchronously | The closure shares a variable that keeps being modified, so the value actually read does not match the intended iteration; late binding of loop variables |
| Cleanup timing at the end of a scope | Deferred execution, destructors, `finally`, `with`, `using`, try-with-resources | Cleanup registered inside a loop does not run until the function ends; the cleanup order is the reverse of what is needed; errors or exceptions in cleanup are swallowed |
| Finalizers and garbage collection | Relying on garbage collection or finalizers to release external resources | Release timing is uncertain; a finalizer closes a handle still used elsewhere; resources run out before collection happens |
| Lifetime of cancellation and context | Cancellation tokens or request contexts stored in long-lived objects; async cancellation | Lifetimes do not match, and cancellation propagation gets out of control; cancellation exceptions swallowed as ordinary errors |
| Exception and error mechanisms | Catch scope, uncaught exceptions, unhandled async rejections, ignored error returns | See "Boundaries of the exception mechanism" in [13](#13-error-handling-and-failure-semantics) |
| Async and event loops | Coroutines, Promise, Future, event loops | Blocking calls stall the event loop; tasks started without keeping a reference get collected or lose their errors; interleaving before and after suspension points (see [18](#18-concurrency-and-memory-model)) |
| Assertions and build modes | Assertions, debug checks, separate debug and release builds | Release builds or optimized modes remove assertions, so assertions used for input validation stop working in production; debug and release builds differ in overflow and bounds checking |
| Strings, bytes and encoding | Conversion, length, indexing, truncation | Bytes and characters mixed up; UTF-16 surrogate pairs split; NUL-terminated strings cut short early; the default encoding depends on the locale; unnecessary copies; the original data changed after a zero-copy conversion |
| Unsafe operations | Every unsafe block, raw pointer and native call | Are the memory layout and lifetime assumptions they rely on written down; visibility to the garbage collector |
| Reflection and metaprogramming | Reflection, dynamic attributes, proxy objects, changing classes or prototypes at runtime | Failed type assertion paths not handled; used on hot paths; cacheable metadata not cached; external input decides which attribute or method is accessed |
| Generics and type erasure | Type parameters, constraints, types that exist only at compile time | Constraints so loose that failures only show up at runtime; zero-value semantics; behavior differs when the type parameter is a pointer vs. a value; type annotations are not checked at runtime but are treated as validation |
| Module loading and initialization | Package or module initialization, side effects at import, dependency cycles, static initialization order | Side effects, failures and execution order of initialization are undefined; a dependency cycle hands out a half-initialized object; the module or library search path can be hijacked |
| Mutable globals | Package-level variables, singletons, module state, class variables | The ownership and concurrency contract of mutable globals is unclear; whether they are allowed is decided by the project rules |
| Language and runtime versions | Declared version, build configuration, runtime switches | Semantics change with the version (loop variables, timers, default encoding, default process start method, etc.); the code is written for the new semantics but declares an old version, or the other way around |

## 17 Time and clocks

| Checkpoint | What counts as a problem |
| --- | --- |
| Wall clock vs. monotonic clock | The wall clock is used to measure intervals → after a clock sync adjustment the interval becomes negative or huge |
| **Clock going backward** | What happens after the clock goes backward to every decision based on absolute timestamps (validity periods, grace windows, throttling, leases, fragment freshness windows): early expiry, never expiring, throttling stuck on forever |
| Precision and granularity | Under a shared coarse-grained clock, events in the same tick cannot be ordered — logic that depends on order breaks |
| Boundary comparisons | Less than or less than or equal; does the behavior on exact equality match the docs |
| Timeout propagation | Is the upstream's remaining deadline passed downstream; a downstream that uses its own fixed timeout makes the total duration uncontrollable |
| Clock skew | Comparing timestamps across machines; how much skew is tolerated |
| Time zones and daylight saving time | Is there a dependency on the local time zone; the repeated hour and the missing hour on the day daylight saving time switches |
| Units | Seconds / milliseconds / nanoseconds mixed up; the unit when converting between duration types and bare integers |
| Timers | Check collection, stop, reset and callback semantics against the actual runtime; after the logic has ended, scheduled tasks still hold resources or change state |
| Injectable | Can tests control time and the order of events; are assertions that rely on sleep or scheduling timing flaky — see [40](#40-testability-and-fault-injection) |

## 18 Concurrency and memory model

First confirm the public concurrency contract and the concurrency paths that are really reachable, then check the items below. Do not treat combinations of operations that callers are explicitly forbidden to perform as implementation defects.

| Checkpoint | How to check | What counts as a problem |
| --- | --- | --- |
| **Data races and logic races** | Tell memory access conflicts apart from interleaving of business operations | Passing a dynamic race detector (Go's `-race`, ThreadSanitizer for C/C++/Rust, etc.) only means no data race was detected in this run; it does not cover paths that were not executed, and it cannot prove that combined operations are correct; for the basis see [Go race detector](https://go.dev/doc/articles/race_detector) |
| **Interleaving under cooperative scheduling** | In single-threaded event loops, coroutines and generators, at each suspension point (await, yield, callback boundary), look for "read before suspending, write based on it after resuming" | Believing "it is single-threaded, so there are no races"; state has already been changed by another task between suspension points |
| Global interpreter lock | Runtimes with a global interpreter lock (GIL) | Treating the global lock as an atomicity guarantee for compound operations (read-modify-write, check-then-act); the assumption fails on builds without a global lock or on other implementations |
| fork and threads | Creating a copy of the process as a child in a multi-threaded process | The child inherits a lock held by another thread and deadlocks; inherited descriptors, random number state and connections are shared by parent and child |
| Concurrency contract | For each public method, confirm whether "can it be called concurrently" is written down | The contract is not written, or is written but the implementation does not meet it |
| State protection | For each field, check the owner, every reader and writer, and the lock, atomic or immutable-publication method used | No protection, or the protection does not cover the read, decision and write that must complete as one unit |
| Check-then-act | Look for the shape "read state → release lock → write back based on it" | The state may have changed in the window in between (e.g. read a value from slow storage → no lock → write it into the in-memory cache, while the key was overwritten or deleted in the meantime) |
| Act first, account later | Look for "change the real data → some code → only then update the count/index" | Observers in that window take the intermediate state as stable (e.g. data already on disk but the count not yet incremented; a scan publishes an absolute count, and then the count is incremented once more) |
| Visibility | What establishes each cross-flow visibility (lock / channel / atomic) | It relies on the intuition that "it should be visible", with no synchronization primitive behind it |
| Combining atomics | Places where two or more atomics must be judged together | Not atomic as a whole |
| Lock granularity and behavior while holding a lock | Is there I/O, a system call, a callback or external code while the lock is held | Holding the lock for long; **calling a callback/interface method while holding the lock**, which may reenter and take the same lock |
| Lock order | List every place that "holds A and takes B" | A reverse order exists → deadlock |
| Reentrancy | Can the same execution flow take a non-reentrant lock twice | Self-deadlock |
| Safe publication | Is an object still changed after it is published | Torn reads |
| Channels and queues | Closing one that is already closed, sending to a closed one, nil channels, blocking on unbuffered ones, dropping messages in non-blocking branches | Crash or silently dropped events |
| Wakeups | Is waking one waiter enough, lost wakeups, does close wake all waiters | Blocked forever |
| Wait groups / task groups | The count incremented inside the child flow, missed done calls, reuse | Wrong counts, early return |
| Background flows | Who starts it, who stops it, does close wait for it, can an error path miss stopping it | Leak |
| Cancellation propagation | After cancellation, do the derived execution flows really exit | Leak |
| Close races | Close running concurrently with every entry point | Writes during close, leftovers after close, counts pushed below zero |
| ABA | A value used in a lock-free compare-and-swap is changed back to its original | Wrongly judged as "never changed" |
| **Lock-free progress** | Compare-and-swap retry loops, spin waits | Retries with no upper bound and no backoff; busy waiting that never yields the processor; execution flows keep making each other fail, forming a livelock |
| **Reused slots** | Reuse in object pools, ring slots, generation pools and free lists | Old references are still treated as valid after a slot is reused; generation numbers are too narrow and collide with old values after wraparound; references that extend an object's lifetime are not cleared before it is returned |
| Starvation and thundering herd | Many waiters woken at the same time; can some kind of operation never get its turn | Tail latency out of control |
| Single-flight | Are repeated expensive operations on the same key merged | Duplicated work, thundering herd |
| Ordering promises | Is FIFO / ordering promised; does the implementation meet it | Promised but not met |
| Clock coarseness | Places that use timestamps to decide order — see [17](#17-time-and-clocks) | Two writes in the same tick cannot be told apart |
| Test synchronization hooks | Wait/idle flags in production code that exist only for tests | Record and explain them; move them into test files if possible |
| Deterministic reproduction | Can a deterministic interleaving be built to reproduce it, instead of running it again and again and hoping to hit it | Only luck → even after a fix, nothing proves it |

## 19 Lifecycle and resources

| Checkpoint | What counts as a problem |
| --- | --- |
| Acquire/release pairing | Created but never released; **early returns on error paths skip the release** |
| Placement and order of deferred release | Registered too late; the execution order of several deferred calls is the reverse of the release order that is needed |
| Manual release | Release written by hand in several places in the same function (miss one and it leaks) — use automatic release where possible |
| **Internal order of close** | When closing, is there a reason for the release order of multiple resources (stop background work before closing handles? wake waiters before clearing state?), and is it written down |
| **Whether close waits** | Does it wait for in-flight operations? For how long? Are the consequences of not waiting (half-finished results, dirty snapshots) written down clearly |
| Reference counting | Increments and decrements do not pair up; a decrement is missed on an error path |
| Repeated close | The behavior does not match the contract, or it releases again a resource already handed over to another object |
| After close | Is the behavior of each entry point defined and consistent — see [12](#12-state-machines-and-transitions) |
| Handles / connections / finalizers | Are there leak paths (for timers see [17](#17-time-and-clocks)) |
| Temporary files and directories | Left behind on failure paths |

> Unbounded growth and whether the bound can be computed are checked in [20 Resource bounds and backpressure](#20-resource-bounds-and-backpressure); memory leak patterns are checked in [31 Performance, memory and latency](#31-performance-memory-and-latency).

## 20 Resource bounds and backpressure

| Checkpoint | What counts as a problem |
| --- | --- |
| Can the bound be computed | Is the worst-case use of memory, execution flows, handles and connections effectively bounded; when this cannot be determined, record the missing conditions, and report it as unbounded only when continued growth is proven |
| Unbounded structures | Queues, buffers, maps, arrays and waiter lists that only grow and never shrink, with no eviction or limit |
| Amplification factor | How many internal operations, allocations and system calls one external request turns into; whose input decides the amplification factor |
| Slow consumers | What happens when production is faster than consumption |
| Backpressure policy | Once full, does it block, drop or return an error — the behavior is not defined |
| Recursion depth | Is the depth bounded; can external input blow the stack |
| Batch entry points | Do batch entry points that accept arguments of any length have a limit; what happens on one huge call |
| **Hash collision flooding** | Untrusted strings used as hash table / map keys, where collisions let a single request use up the CPU |
| Algorithmic cost | Can external input trigger regex backtracking, combinatorial search or other superlinear computation; judge by the actual algorithm and engine, and do not report denial of service based only on the shape of an expression |
| Concurrency limit | Is there a limit on operations in flight at the same time; who controls it |
| Quota ownership and accounting | Are limits bound to a trusted account or tenant; can switching addresses, batching requests or switching nodes bypass them; is budget reserved before expensive operations, and settled correctly after cancellation and failure |

## 21 Scalability and capacity

> The previous dimension asks "will it be overwhelmed". This one asks "**what happens at ten times the size, and does adding machines help**".

| Checkpoint | What counts as a problem |
| --- | --- |
| Growth dimensions | Data volume / concurrency / connection count / tenant count / request rate — do you know which one hits the bottleneck first |
| **Tenfold growth** | What breaks first at ten times the current scale; does it degrade linearly or fall off a cliff |
| Linear scaling | Does adding instances / shards raise capacity linearly; is there a global serialization point that cancels it out |
| State and scaling | Is there local state that blocks horizontal scaling; can it be shared-nothing |
| Sharding and rebalancing | The choice of shard key and hot spots; the cost of data migration and rebalancing when scaling out |
| Uneven hot and cold load | Hot keys, hot partitions, long tails; is there a way to spread them out |
| Single points | Are there single points (global locks, single-instance components, a sole sequence number generator, a single writer) |
| Capacity headroom and alerts | How much headroom is left now; is there an alert when nearing the limit |
| Backpressure passed upstream | When capacity is reached, can the pressure be passed back upstream correctly instead of piling up locally |
| Startup amplification | Can many instances starting / reconnecting at the same time take down downstream services (thundering herd, all caches empty) |

## 22 Fault isolation and degradation

> The previous dimension asks "can I hold up on my own". This one asks "**when others break, what do I do**".

| Checkpoint | What counts as a problem |
| --- | --- |
| Behavior when a dependency fails | What happens when each external dependency fails: hard failure, degradation, default values, stale cache — is it defined |
| Circuit breaker | Is there a circuit breaker; the transition conditions of its state machine (closed / open / half-open), the recovery conditions, the probe volume while half-open |
| Bulkhead isolation | Can one class of requests using up resources drag down another; are pools, queues and execution flows separated by class |
| **Timeout budget allocation** | How the total budget is split across stages; is the downstream timeout smaller than the upstream's remaining time; can the timeouts of serial hops add up to more than the total budget |
| **Retry storms** | Do retries at several layers multiply the calls; are budget, backoff and jitter effective; can side effects that already happened or whose result is unknown be retried safely; for the idempotency criteria see [23](#23-transactions-atomicity-and-consistency) |
| Observable degradation | Is anyone told when degradation happens; can you see whether it is in a degraded state right now |
| Automatic recovery | Does it recover by itself after the dependency recovers; is manual intervention needed; does recovery cause a thundering herd |
| Partial availability | When some features break, do the others still work, or is everything down |
| Slow is worse than down | What happens when a dependency gets slow (not failing) — can timeouts catch it; do connections / execution flows get used up |
| Cache as degradation | When degrading to stale data, how stale is too stale to use; is the caller told the data is stale |

## 23 Transactions, atomicity and consistency

> This dimension asks "**which things must become true together**". Transactions are not only for databases: any code that "changes two places" must answer these questions.

| Checkpoint | What counts as a problem |
| --- | --- |
| Unit of atomicity | Which operations must succeed or fail together; where the boundary is drawn; is it written down |
| Transaction boundaries | Where the transaction opens and where it closes; is there a rollback on error paths; is there a long transaction that wraps slow operations inside it |
| Nesting and reuse | The semantics of nested transactions; the same transaction committed separately by several layers |
| Isolation level | Which level is used; can dirty reads / non-repeatable reads / phantom reads affect correctness |
| **Consistency promises** | Strong consistency / eventual consistency / read-your-writes / monotonic reads — what is promised, does the implementation match it, where is it written |
| Cross-resource operations | How are operations across two stores / two services / a store plus a cache kept atomic; is there compensation |
| Compensation and rollback | What if the compensation itself fails; is the compensation idempotent; can the compensation clash with the normal path |
| Dedup and idempotency keys | How repeated requests are recognized; where the idempotency key comes from; how long the dedup window is; what happens to repeats outside the window |
| **"Exactly once"** | Where exactly-once is claimed, is the real semantics "at least once + an idempotent consumer"; is this stated clearly |
| Visibility order | Is there a promise on the order in which writes become visible to other readers; is the cache written first or the store |
| Read and write paths differ | Writes go one way and reads go another (such as write to the store, read from the cache); how long is the window in between |

## 24 Encoding and persistent formats

| Checkpoint | What counts as a problem |
| --- | --- |
| Self-description | Are the existing format markers, schema or external constraints enough to identify the data; foreign or unsupported formats get processed as valid data |
| Integrity checks | No length or checksum, so truncation cannot be detected; does the check cover all of the content |
| Reserved fields | Not zeroed on write; not checked on read |
| Byte order | Inconsistent from place to place |
| Untrusted input | See [4.3](specialties.md#43-decoding-untrusted-external-data) |
| Identity stability | For the key → storage location mapping, can a config change make it hit old data; is the order of the inputs that produce the mapping stable (see [15](#15-algorithm-and-data-structure-correctness)) |
| Foreign data | Can someone else's data in the same location be wrongly deleted, read or counted |
| Local corruption | Can one bad record keep the whole thing from opening — can it skip it and go on |
| **Round trip and canonical form** | Decoding and then re-encoding is not equivalent to the original data; unknown fields are silently dropped, or passed through as-is without the contract saying so; when one value has several encodings, a non-canonical form is accepted while signatures, hashes, dedup or cache keys depend on the canonical form |
| Version compatibility | See [26](#26-release-upgrade-migration-and-rollback) |

## 25 Crash and recovery

Tell the actual faults apart: normal shutdown, exceptions, graceful termination, forced kill, power loss, out of memory and storage failure. Forced kill and power loss cannot rely on process cleanup; whether cleanup can still run when a container stops or storage turns read-only must be checked against the real signals and error paths. Recovery guarantees follow the fault scope the project promises.

| Checkpoint | How to check | What counts as a problem |
| --- | --- | --- |
| Classifying termination modes | For each termination mode, ask "which line can still run this time" | Only normal shutdown is considered; recovery relies on things that only the normal path writes |
| **Durability** | When durability across power loss is promised, check the file, directory entry and commit syncs the target platform needs, and how sync failures are handled | A cached write or an atomic replace is treated as a durable commit; data can still be lost on power loss after success is returned |
| Enumerating crash points | Along the write path, ask **line by line**: "if power is lost here, what is the state outside the process? Can the next open tell?" | An intermediate state exists that cannot be determined |
| Write order | Log first, then data? Can the order support recovery? Can the lower layer reorder it | If the order assumption does not hold, the whole recovery fails |
| Partial writes | A write may only get halfway (short write); torn writes at sector/page granularity | Relying on the assumption "one write is atomic"; not checking the byte count a write returns |
| Publication atomicity | Writing the final state directly vs. a temp file plus an atomic replace; is the cost of each written down | The cost of writing directly (a failed overwrite loses the old data) is not in the docs |
| Recognizing half-finished data | Use write ownership, commit markers, length and integrity checks to tell in-flight writes apart from leftover fragments | Time or length alone cannot prove the content is fully committed; recovery wrongly deletes data that is still being written, or accepts damaged data of the same length |
| Recovery coverage | Does recovery cover every intermediate state, or only the few that someone thought of | Holes |
| **Crash during recovery** | Recovery crashes again halfway through | Recovery is not idempotent → the second open is worse |
| Idempotent replay | Does replaying the log twice give the same result | Not idempotent |
| Recovery cost | How much must be scanned after a restart; how the time grows with the data volume | At the scale it claims to support, startup needs a full scan |
| External interference | Someone deleted the directory / changed the data / the mount point disappeared / it became read-only | Crashes outright, or silently loses data |
| Resource exhaustion | Disk full, inodes used up, handles used up, system calls interrupted | Not handled, or after handling, the in-memory state and the persistent state disagree |
| **Multiple processes** | Check the transactions, conditional writes or lock mechanism for shared writes, and how they are released after a crash | Without effective coordination, processes overwrite each other, or leftover locks block recovery; do not call it an issue just because there is no file lock |
| In-process exclusivity | The same process opens the same instance twice | Two sets of state overwrite each other |
| Backups and drills | Are there backups; **has the recovery flow been drilled**; are the backups themselves usable | Backups exist but have never been verified to restore |

## 26 Release, upgrade, migration and rollback

| Checkpoint | What counts as a problem |
| --- | --- |
| Forward compatibility | What happens when an old version reads data written by a new version — an error, a skip, or a **silent misread** |
| Backward compatibility | Old data that the project still needs to read cannot be read, and there is no agreed migration or explicit rejection |
| Version markers | No version field in the format; no explicit handling when an unknown version shows up |
| Migration | Is the migration idempotent; can it resume after crashing midway; can a failed migration go back to the original state; how long the migration takes and what it locks |
| **Release strategy** | Gradual / canary / batched rollout; how long the observation window is; what triggers an automatic rollback; the risk of shipping everything at once |
| **Rollback mechanism** | Is rollback a single command; how long does it take; what happens while old and new versions coexist during the rollback |
| Data after rollback | After upgrading and then rolling back to the old version, is the data still usable; is it a one-way door (if so, write it down and confirm before release) |
| Mixed-version operation | When the release method lets old and new versions run side by side, is it safe for them to share data and talk to each other; are the prerequisites for a switchover with downtime actually in place |
| Feature flags | The default value, scope and cleanup plan of each flag; have combinations of several flags been verified |
| Breaking interface changes | Have callers migrated per the confirmed plan; whether compatibility is needed is decided by the project contract; do not add a transition layer automatically |
| Configuration migration | Does the meaning of old config change in the new version; is direct replacement or a transition strategy consistent with the authorization |
| Dual-write / dual-read period | Check only when this approach has been adopted: how inconsistencies between old and new data are handled, and when the dual track stops |

## 27 Security and trust boundaries

| Checkpoint | What counts as a problem |
| --- | --- |
| **Classifying input sources** | Callers / config / **data already persisted** / **network peers** / environment variables / command line each need their own trust level. Treating stored data or peer data as trusted is a common mistake |
| Input validation | Validation is missing, is done on a different representation from the one actually used, or its constraints are lost because the data is decoded, concatenated or normalized again after validation |
| Injection surface | Check by the interpreter the data finally enters: for queries see [4.7](specialties.md#47-databases-and-queries), for text output see [4.17](specialties.md#417-templates-and-text-output), for execution see [4.22](specialties.md#422-subprocesses-dynamic-execution-and-decoder-side-effects) |
| Second-order use | Data that was safe when first saved loses its protection when later concatenated into a query, command or template; coming from a database or message queue does not exempt it from validation |
| Parser differentials | The gateway, auth layer and handler interpret duplicate parameters, duplicate JSON fields, encodings or type conversions differently, so the value that is checked differs from the value that is executed |
| Case and normalization | A path normalized to lowercase makes two different directories the same one on a case-sensitive file system; locale-dependent case conversion |
| Paths and local permissions | All covered in [4.8](specialties.md#48-file-systems-and-paths); also check whether the resource can be swapped after validation |
| Cryptographic controls | Hash purpose, randomness, credentials and cryptographic side channels are all covered in [4.4](specialties.md#44-cryptography-and-credentials) |
| **Information leaks** | Error pages / debug endpoints / version and stack information / directory listings / verbose response headers exposed to the outside |
| **Hardening** | Default accounts and default passwords; unneeded features, ports and endpoints turned on by default |
| Denial of service | Check the cost and amplification an attacker can trigger; for resource budgets see [20](#20-resource-bounds-and-backpressure), for decoding see [4.3](specialties.md#43-decoding-untrusted-external-data) |
| Secure by default | Secure defaults are all covered in [32](#32-configuration-and-defaults); for which way to fail see [29](#29-fail-safe-behavior-and-dangerous-operations) |

## 28 Authorization and access control

> [27](#27-security-and-trust-boundaries) asks "can this input be trusted". This dimension asks "**is this requester allowed to do this**". The two are often confused, and then authorization gets missed.

| Checkpoint | What counts as a problem |
| --- | --- |
| Separating authentication and authorization | "Who you are" and "what you can do" are decided together |
| **Where checks happen** | Is the permission check done at every entry point; are there bypass paths (internal methods called directly, a batch entry point that checks only the first item) |
| Default deny | Do unknown cases, unknown roles and new entry points default to deny or to allow |
| **Horizontal privilege escalation** | Can changing a resource identifier give access to other people's objects; a hard-to-guess identifier does not mean there is an authorization check |
| **Vertical privilege escalation** | Can low privileges reach high-privilege operations; does a hidden entry point count as protection (it does not) |
| Field-level authorization | Some fields in the same object should not be seen / changed by some people; is that enforced |
| Automatic field binding | When a request binds directly to an internal model or does a bulk update, can it write server-managed fields such as role, tenant, owner or balance; does the response expose attributes the requester may not read |
| Consistency across entry points | Do lists, search, export, nested objects, batches, GraphQL fields, RPC and async tasks enforce the same object authorization; checking only the detail endpoint is not enough |
| Privilege escalation paths | Are there paths that combine, step by step, into higher privileges |
| Source of authorization data | Does the permission decision rely on a trusted source, or on fields the request itself carries |
| Granularity | Is the permission granularity fine enough; is it "all or nothing" |
| Caching and invalidation | After permissions, tenant relationships or resource ownership change, how long until existing sessions, long-lived connections, background tasks and caches are bound by the new permissions; are the authorization conditions still met right before execution |
| Multi-tenant boundaries | Are data, resources and quotas isolated between tenants; do cross-tenant queries have a mandatory filter |
| Delegation and acting on behalf | When acting on behalf of someone else, is the action bound to the original principal, the target resource and the allowed operations; the service's own high privileges cannot stand in for the caller's authorization |

Security records for sensitive operations are all covered in [4.23 Security audit logs](specialties.md#423-security-audit-logs).

## 29 Fail-safe behavior and dangerous operations

> On error, does the system stop on the **safe side** or on the dangerous side. This dimension matters most for code that "changes external state".

| Checkpoint | What counts as a problem |
| --- | --- |
| **Failure direction** | On error, deny or allow by default; when the auth / rate limiting / validation component is down, does it block everything or let everything through |
| Confirming dangerous operations | Do irreversible operations such as delete, overwrite, migrate and reset have a second confirmation / preview / dry run |
| Irreversibility | Which operations cannot be undone, and is that written down; is there soft delete or a retention period as a safety net |
| **Limiting the scope of impact** | Do dangerous batch operations have a scope limit; **does it refuse to run when there is no limiting condition** (e.g. a delete with no filter) |
| Dry runs and rehearsals | Is there a mode that only reports and does not execute; can the rehearsal result differ from the real run |
| Operation preconditions | Are there constraints such as "must not run under these conditions" (no backup, version mismatch, not enough space, unexpected target); are they checked |
| Risk warnings | Do dangerous paths give explicit warnings in the docs, names and logs |
| Blast radius | How much can one mistake destroy at most; is there batching, rate limiting, a circuit breaker |
| Non-idempotent destructive retries | What happens when a non-idempotent destructive operation is retried |
| Recovery path | Is there a way back after a mistake; has the way back been verified |

## 30 Privacy, data governance and compliance

| Checkpoint | What counts as a problem |
| --- | --- |
| Data classification | Which fields are personal / sensitive information; are they labeled |
| Minimization | Is what is collected and stored actually needed; is there any "keep an extra copy while we are at it" |
| Retention period | How long data is kept; is it deleted when it expires; where the period comes from |
| **Whether deletion is real** | After deletion, is it still in backups, logs, caches, indexes, derived data or message queues; is soft delete treated as real deletion |
| Anonymization | Is anonymization / pseudonymization reversible; can people be re-identified through linked fields |
| Cross-border transfer and region | Region limits on where data is stored and where it is sent |
| Logs and telemetry | Is there personal information or credentials in logs, metric labels, traces or crash reports |
| Purpose limitation | Does the use go beyond the scope declared when the data was collected |
| Third-party sharing | Does data passed to third parties (including libraries, SDKs and external services) go beyond what is needed |
| Encryption requirements | Are the encryption requirements for data in transit and at rest met |
| Source of requirements | Have the applicable regions, data categories, contracts or organizational policies been confirmed; when the source and version are unclear, record the requirement as pending confirmation, and do not write technical check results up as legal compliance certification |

For the completeness and retention policy of access records for sensitive data, see [4.23](specialties.md#423-security-audit-logs).

## 31 Performance, memory and latency

| Checkpoint | How to check |
| --- | --- |
| Hot-path allocation | Look for escapes and temporary objects: boxing, closures, dynamic growth, a new buffer on every call, string concatenation, deferred calls or exceptions on extremely hot paths |
| Unneeded copies | Defensive copies where immutable data could be shared (or the other way around: a copy that is needed but not made) |
| System call count | How many system calls one logical operation makes; can any be short-circuited |
| Lock contention | Global locks and coarse-grained locks on hot paths |
| Complexity and scale | Does the algorithmic complexity fit the data scale it claims to support |
| Preallocation | The length is known but capacity is not reserved |
| Cache lines | False sharing among fields that are hit concurrently at high frequency |
| **Memory leak patterns** | Long-lived containers hold short-lived objects; closures accidentally capture large objects; listeners / callbacks are registered and never unregistered; caches have no eviction |
| **Object pools** | Is an object still used after being returned; is the reset of pooled objects complete (data left over across requests is a security issue) |
| Large objects and fragmentation | How often large blocks are allocated; fragmentation from objects that stay resident for a long time |
| **Tail latency** | Only averages are looked at, not high percentiles; sources of jitter (locks, garbage collection, retries, cache misses) |
| Cold start and warm-up | The extra cost of the first call; is warm-up needed, and is it done |
| Runtime pauses | Pauses from garbage collection / compaction; how the allocation rate affects pauses |
| Benchmarks | Are hot paths covered by benchmarks; are there comparable numbers before and after a change; are allocation counts looked at |

**Discipline for reporting performance conclusions**: any cost you give is either measured, or has the basis of its estimate written out (how many system calls, how many atomic operations, the order of magnitude against a baseline). **Never present a number you did not measure as a measured one**.

## 32 Configuration and defaults

| Checkpoint | What counts as a problem |
| --- | --- |
| Default values | Unreasonable, undocumented, or inconsistent with the docs; is the default configuration safe |
| Zero value vs. unset | The two need to be told apart but are not ("an explicit 0 means off" vs. "not set means use the default") |
| Hard-coded policy | A configurable policy is written into the code. **Those changed in this change, or directly related to it, must be flagged to the caller**; out-of-scope ones are only reported |
| Validation completeness | Some fields are not validated; validation rules do not match the docs / declared tags; combined validation (A must be less than B) is missing |
| Validation timing | Validated at startup, or only blows up when used |
| Merge semantics | Are the rules for picking values when several configs are merged/inherited (take the larger, the smaller, the first) written down clearly |
| Source precedence | Command line / environment variables / files / defaults — which overrides which; is it written down |
| Units | Sizes and counts mixed up; the range of supported unit suffixes |
| Config that affects identity | Which config items take part in the instance identity, so that changing them means a different set of data — is this written down |
| Secret configuration | Does config loading expose secrets or wrongly inherit permissions; credential exposure and lifecycle are all covered in [4.4](specialties.md#44-cryptography-and-credentials) |
| Hot reload | Can it be changed while running; how a change affects in-flight operations and existing connections; can a bad change be rolled back |

## 33 Observability and diagnosability

| Checkpoint | What counts as a problem |
| --- | --- |
| Log placement and frequency | Logging on hot paths; flooding the log on failure; logging cannot be turned off or have its level changed |
| Log and error content | Leaking keys, paths, credentials or user data — see [30](#30-privacy-data-governance-and-compliance) |
| Locatable errors | The operation type, safe resource identifiers and correlation information are missing, so failures cannot be located; the needed context must still meet the data minimization requirements of [30](#30-privacy-data-governance-and-compliance) |
| Correlation IDs | Can things be tied together across components / requests (request ID, trace ID); are the IDs passed all the way down |
| Exposed statistics | Are the exposed counts/sizes/states **exact or estimates**; are the error sources and convergence conditions written down |
| Estimates used as exact | The docs say it is an estimate, but callers (and tests) use it as exact |
| Key paths visible | Are the few numbers you most want when something goes wrong (queue length, in-flight count, failure rate, latency percentiles) exposed |
| **Service objectives** | Are there explicit availability / latency targets; can they be measured; who finds out when they are missed |
| **Alerts** | Do critical failures have alerts; are the alerts actionable, are they noisy, can they miss things |
| **Ownership and runbooks** | Who owns this; where are the steps for handling problems written; can a newcomer follow them |
| Self-check entry points | Is there a way to run a consistency check; can internal state be verified when something goes wrong |
| On-site information | Is the information you get when something goes wrong enough to reconstruct what happened |

## 34 Runtime environment and deployment contract

| Checkpoint | What counts as a problem |
| --- | --- |
| Environment assumptions | Are the assumptions about directory existence, permissions, network reachability, time zone, locale and temp space written down; when they are not met, is there a clear error or a strange failure |
| Environment variables | When they are read (at startup vs. on each use), behavior when missing, default values, case and naming conventions |
| **Awareness of resource limits** | Are concurrency, memory pools and buffer sizes computed from the actual quota or from the host's total — computing from the total inside a container badly oversubscribes. Whether the runtime is aware of container limits varies by language and version; see [Appendix A](languages.md) |
| Container specifics | Signal forwarding and zombie reaping when running as PID 1; read-only root file system; where the writable directories are; the size of the temp directory |
| Handles and ports | Where the limits come from; what happens when they are exceeded |
| Startup dependencies | When a dependency is not ready, does it retry and wait or fail right away; is there an implicit assumption about startup order |
| Shutdown contract | How long it is allowed to take after a termination signal; it gets force-killed on timeout; does that duration match the internal shutdown timeout |
| Clock and localization | Does it depend on the host time zone / locale; can behavior differ across hosts |
| Single-instance assumption | What happens when several replicas run at the same time; is there an implicit "I am the only one" assumption (local locks, local caches, local counts) |
| Storage assumptions | Is the local disk persistent; is it still there after a restart; do replicas share the same storage |
| Runtime identity and isolation | Does it use system or cloud permissions beyond its duties; do container privileges, host mounts, runtime sockets, service accounts and secret mounts widen the impact after a compromise |
| Network and management plane | Do the actual listen addresses, management interfaces, debug interfaces, object storage public policies and ingress rules match the deployment contract; do not assume that being "on the internal network" means no identity or permission checks are needed |
| Configuration drift | Do the artifacts actually deployed and the config actually in effect match the review input; do environment overrides, side entry points or security config that is not enabled change the conclusion |

## 35 Dependencies

| Checkpoint | What counts as a problem |
| --- | --- |
| Necessity | The standard library or the project already has an equivalent capability |
| Usage | A whole library pulled in for one small function |
| Understanding the contract | Are the assumptions about the dependency's behavior guaranteed by its docs; does the code rely on its undefined behavior |
| Failure modes | Behavior when the dependency fails / gets slow — see [22](#22-fault-isolation-and-degradation) |
| Upstream forks | Is the boundary still "upstream mirror + minimal local patches"; are the patches still needed, is there a public alternative; does the sync method still work |
| Transitive dependencies | Size, licenses and conflicting versions of transitive dependencies |

> The trustworthiness of dependencies, lock file integrity and vulnerability scanning are checked in [36 Supply chain and artifact integrity](#36-supply-chain-and-artifact-integrity).

## 36 Supply chain and artifact integrity

| Checkpoint | What counts as a problem |
| --- | --- |
| Trusted sources | Are the dependency sources trustworthy; are there impostor packages with look-alike names; is anything pulled from unofficial channels |
| **Dependency confusion** | Internal package names can also be resolved from public registries; is the build locked to use only internal sources |
| Locks and checksums | Is the lock file committed; is there checksum verification; can dependencies be swapped silently |
| **Known vulnerabilities** | Judge whether the target is affected from the actually locked versions, transitive dependencies, fork patches, build conditions, reachable paths and vendor advisories; record the tool and the date of the vulnerability database; a package-name match is not the same as an exploitable vulnerability, and an empty result is not proof of safety. By default, query public vulnerability databases and official advisories online (allowed scope in [the target project's hard boundaries](scope.md#the-target-projects-hard-boundaries)); without network access, state the date of the offline database and what could not be checked |
| **Security fix versions** | The locked version is behind a security fix already released upstream; check the registry and release notes for the lowest version with the fix, the latest supported version, and whether upgrading crosses a breaking change |
| Maintenance status | Is the dependency abandoned, unmaintained for a long time, run by a single maintainer, or recently handed over to someone new |
| Reproducible builds | Does the same input produce the same artifact; are timestamps, absolute paths or build machine information embedded |
| Version traceability | Where does the version marker in the artifact come from; can it be matched to the source |
| Signing and distribution | Are artifacts signed; are the distribution channels trustworthy |
| Code execution at build time | Can the build run arbitrary code from dependencies (install scripts, generation steps, plugins) |
| Build and release permissions | Can untrusted commits, dependency scripts or workflow inputs read release credentials, write artifacts or deploy; are build jobs, runners and release identities isolated from each other |
| Build cache and artifact substitution | Can untrusted jobs poison caches or intermediate outputs that trusted jobs use; is the identity and digest checked in the final signature verification bound to the artifact actually deployed; can tags or download URLs be swapped |
| Component inventory | Can the runtime, base images, dependencies and plugins be traced from the actual artifact; does the existing software bill of materials (SBOM) match the artifact; do dynamic downloads bypass locking and integrity checks |
| Licenses | Are dependency licenses compatible with the way this project is distributed; are the notice files complete |

## 37 Interoperability and coexistence

> One is "can it talk to others", the other is "can it get along with others on the same machine".

| Checkpoint | What counts as a problem |
| --- | --- |
| Standards conformance | Do the formats, protocols and encodings exchanged with the outside conform to public standards; are deviations recorded with reasons |
| Version negotiation | Version / capability negotiation with the peer; behavior on a mismatch; can a downgrade in the negotiation be forced |
| Liberal in, strict out | Tolerant of data received, strict about data sent — is the balance right; can it turn into hiding problems |
| Interop with other implementations | Has it only been tested against its own implementation, or also verified against third-party implementations |
| **Resource coexistence** | When it shares resources (ports, directories, lock files, caches, temp space, shared memory) with other processes / instances on the same machine, can they interfere with each other |
| **Namespace collisions** | Can global names (environment variable names, file names, metric names, registration keys, lock names) collide with someone else's |
| Side effects spilling over | Can changing shared system state (network config, system settings, global singletons) affect others; is it restored on exit |
| Multiple versions side by side | What happens when several versions of the same dependency, or of this program, are present at the same time |
| Yielding resources | Is there a timeout and a way to yield when holding shared resources; can it hold them exclusively for a long time |

## 38 Portability and build context

| Checkpoint | What counts as a problem |
| --- | --- |
| Paths | Hard-coded separators; functions for file system paths and URL paths mixed up |
| File system differences | Case sensitivity, permission models, atomic replace guarantees, maximum file name length, link support, file name normalization forms → one logical name may map to two files |
| Word size | Relying on platform integers being 64-bit; **overflow branches that were unreachable become reachable on 32-bit platforms**; for atomic alignment see [16](#16-language-and-runtime-pitfalls) |
| Byte order and alignment | Relying on a specific byte order; assumptions about struct layout |
| System call and network stack differences | Differences across platforms in socket options, port reuse, routing and interface enumeration, and name resolution paths |
| Build conditions and native code | Build tags, conditional compilation macros, platform-suffixed files, optional dependencies and feature switches (such as optional install groups and compile features) give the same source several build results; static analysis only covers the current platform and the default conditions, so **entry points in other build contexts must be stated separately as not covered** |
| Cross-compilation | Does it build for every platform it claims to support |
| Platform-specific code | Do not judge it dead or rewrite it just because the current environment cannot verify it; when the task really involves that behavior, change it within the authorization and state which platforms were not verified |
| **Standalone build context** | Repo-level workspaces and local overrides (such as Go workspaces and `replace`, npm/pnpm/yarn workspace and link, pip editable installs, Cargo `[patch]` and path dependencies, CMake subdirectories) hide what is missing when a unit is built alone; can units that will be published or referenced from outside build and test outside the workspace |
| Minimum runtime version | The declared minimum language, runtime or compiler version is lower than what the syntax, standard library or semantics the code actually uses require; or the declared version keeps new semantics from taking effect |

## 39 Generated artifacts and toolchain

| Checkpoint | What counts as a problem |
| --- | --- |
| Freshness of generated artifacts | Generated artifacts lag behind the generation source |
| Hand-edited artifacts | Someone edited a generated file directly, and it will be overwritten |
| Reproducible generation | The generator version is not pinned, so different machines produce different artifacts |
| **Maintenance boundaries** | Are the generation source, generated artifacts, upstream mirrors and local patches kept apart; writes, docs and verification of affected paths all follow the project rules; do not mechanically require scanning or testing the whole upstream code |
| The generator itself | When the target contains generators, macros or build-time plugins, review them per [4.31](specialties.md#431-code-generators-and-build-time-tools) |

## 40 Testability and fault injection

> This item does not review "are the tests well written" but "**is there any way to test this code**".

| Checkpoint | What counts as a problem |
| --- | --- |
| Dependency injection points | Clock, file system, network, random numbers and external services are hard-coded as globals, so behavior cannot be controlled |
| Fault injection | Storage errors, short writes, dropped connections, partial failures, out of space and dependency timeouts cannot be produced → error paths can never be tested |
| Deterministic interleaving | Concurrency issues can only be hit by running again and again and hoping; there is no deterministic way to reproduce them |
| Observable results | The claimed behavior cannot be verified through return values, state or side effects; when public results can verify it, do not call it an issue just because private fields cannot be seen |
| Environment dependencies | It cannot be tested without a real database / network / privileges |
| **Cost constraints** | Prefer building branches from existing inputs and dependencies; whether internal changes are allowed follows the project rules; do not add unrelated abstractions, public switches or hot-path cost for tests; when it cannot be verified reliably, state the boundary |

## 41 Test quality

| Checkpoint | How to check | What counts as a problem |
| --- | --- | --- |
| Always-true assertions | Static analysis + manual reading | Checks that can never fail |
| Tests of implementation details | Read what the assertions target | Only the internal implementation is verified, not the external behavior |
| Redundancy | Compare the behavior the cases cover | Several cases verify the same branch of the same behavior |
| Unused helpers | Static analysis | Test helper functions that nobody calls |
| Assertion precision | Read the assertions | Only checks "there is an error", not which error; only checks "there is a value", not whether the value is right |
| Flakes | First read the existing failure evidence and synchronization conditions; any runs needed follow the project test strategy | Intermittent failures are hidden by reruns, or the failure conditions are not recorded |
| **Can the test really catch the bug** | Compare against the triggering input, the key assertions and the evidence from before the fix; for isolated comparison see [Execution boundaries during review](scope.md#execution-boundaries-during-review) | The case passes, but it does not reach the issue path, or its assertions do not check the actual effect |
| **Assertions outside the test's execution context** | Search for threads, coroutines, callbacks, unawaited Promises or tasks started in tests | In many frameworks, a failed assertion there does not fail the test, or is only reported after the test ends; some frameworks require "fail immediately" to be called only from the test's own execution flow (such as Go's `FailNow`), so child flows must use "record the failure and return" instead |
| **Test double drift** | Check fakes, mocks and stubs against the contract of the real dependency | The double is looser than the real dependency or behaves differently (no errors, no timeouts, fixed order, no concurrency, no argument checks), so tests are all green but the real environment fails |
| Test method vs. issue type | Check by target type | Parsers and decoders lack fuzz corpora; engines or protocols with a reference implementation lack differential comparison; algorithms lack property tests; data exchanged with external implementations lacks interop samples. Only report the gaps; whether to add or run tests follows the project test strategy |
| Parallelism and cleanup order | Read parent and child tests | The parent test's deferred cleanup may run **before** the parallel child tests; shared resources should use the cleanup hooks the framework provides |
| Parallelism and shared state | Do parallel cases share directories, global registries or ports | They interfere with each other |
| Leaks | Execution flows, temp files, connections and background tasks left after a case ends | Leak |
| Randomness and seeds | Do randomized cases use a fixed seed | Failures cannot be reproduced |
| Environment dependencies | Network, real paths, fixed sleeps, the current time, time zone | Not reproducible / slow / flaky |
| Benchmark correctness | Was the result optimized away by the compiler; was setup work counted in the timing | The numbers mean nothing |
| Scale | Has it been verified at a real order of magnitude (assertions inside a large-data benchmark are a cheap way) | Tested on only a few records, yet claimed to support very large scale |
| Race detection | Check the paths, interleavings and tool records of the concurrency verification | Interleavings that never ran, or logical atomicity, treated as proven; for the limits of the conclusion see [18](#18-concurrency-and-memory-model) |
| Negative cases | Look for assertions that "what should not happen really did not happen" | Only the success path is tested |
| Side effects after rejection | Check the failure assertions for unauthorized access, invalid signatures, illegal input and the like | Only the error code is checked, with no check that data was not written, quota was not deducted, external calls were not made or sensitive fields were not returned |

For the maintenance scope of tests, see [39](#39-generated-artifacts-and-toolchain).

## 42 Coverage

Analyze uncovered behavior in scope only when there is a valid coverage record, or collecting one has been approved for this run. Coverage only describes what was executed; it cannot prove that assertions are effective or that the code is secure. When there is no record, say so plainly, and do not generate a full baseline on your own.

| Category | Handling |
| --- | --- |
| Testable but not tested | A read-only review reports the behavior gap; when changes are authorized, add tests for the behavior this change affects, per the project test strategy |
| Unreachable | Handle it under [dead code](#1-dead-code-and-reachability); do not write tests for it |
| Reachable only with injected faults or on specific platforms | Record it as an unverified boundary and give the reason; for whether faults can be injected, see [40](#40-testability-and-fault-injection) |

Give priority to real success, rejection and failure semantics; for the cost constraints on implementing tests, see [40](#40-testability-and-fault-injection).

## 43 Documentation consistency

| Checkpoint | How to check |
| --- | --- |
| Do the documents exist | Per the project rules, do the directories that should have an overview document (README) / flow document / test matrix have them |
| Entry point mapping | Per the authorization, use an existing targeted tool or check by hand; entry points in scope should have no missing, duplicate or extra mappings |
| Flow accuracy | **A passing tool does not mean the flow is drawn right**: for each entry point, compare it with the source by hand — how parameters affect branches, whether every exit is drawn, whether the source and pass-through rules of dynamic errors are written down |
| Invariants | Does each one state its owner, the conditions under which it holds, what it guarantees, the result of breaking it, and the related flows |
| Stale facts | The docs mention things that were deleted, never implemented or renamed |
| Single authoritative location | Is the same fact in two places that both only point elsewhere, so it is never really written down anywhere |
| Are contracts written down | Concurrency contracts, caller obligations, consistency promises, failure semantics, resource ownership, security assumptions — are these, the things most in need of writing down, actually written |
| Example code | Do the examples in the docs still compile/run; do they use entry points that were deleted |
| Test matrix | Do the case names in the matrix and in the code match in both directions |
| Links and renderability | Do relative links, anchors and cross-references point to files and headings that exist; can diagram syntax (such as mermaid) be parsed and rendered |

> The quality of the comments themselves is checked in [6 Naming, comments and readability](#6-naming-comments-and-readability).

## 44 Project rule compliance

Check item by item against **the target project's own rules**. Cover at least these categories; the specific clauses follow that project's rules files:

| Category | What to check |
| --- | --- |
| Directories and responsibilities | Is new content placed where its responsibility belongs; were other modules changed out of bounds |
| Change scope | Does it go beyond the scope authorized this time; are out-of-scope issues only reported and left untouched |
| Maintenance boundaries | Does it respect the generation and upstream boundaries in [39](#39-generated-artifacts-and-toolchain), and the platform verification limits in [38](#38-portability-and-build-context) |
| Naming and style | Naming rules, comment rules, formatting rules |
| Compatibility layers | When an old implementation is replaced, are the old names fully removed |
| Tool and command boundaries | Do commands, reads, writes and external access follow the project rules and this authorization; the examples in this checklist cannot stand in for authorization |
| Caches and temp directories | Do their location and naming follow the rules |

## 45 Long-running and resident processes

> Most earlier dimensions look at "one call" or "one burst". This dimension looks at two cases: **when there is nothing to do**, and **after running for a long time**. It applies to services, daemons, resident clients, long-lived connection components and long-running batch jobs.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Idle overhead** | Periodic wakeups, polling, full scans or overly frequent heartbeats even with no requests; the wakeup frequency is out of proportion to the real workload; the ongoing cost on battery-powered devices or in pay-per-use environments |
| **Hot loops on persistent errors** | A local error that persists (already exists, permission denied, corrupt config, dependency unreachable) makes retries or the event loop spin without backoff, flooding logs and metrics along the way |
| Self-triggering | Its own actions trigger itself again: watching files it writes, capturing traffic it sends, changing config inside a config-change callback |
| **Wraparound of counters and IDs** | Can counters, sequence numbers, generation numbers, connection or session IDs and auto-increment primary keys wrap around while running; after wraparound, do they collide with old values still in use (same root cause as ABA in [18](#18-concurrency-and-memory-model)) |
| Slow growth | Growth that only shows after days or weeks: caches, mapping tables, log files, temp files, handles, listeners, metric series; is there rotation, eviction or a bound (for bound criteria see [20](#20-resource-bounds-and-backpressure)) |
| **Expiry while running** | Certificates, public key pins, tokens, root certificates, date constants and API deprecation deadlines of dependencies that are embedded or loaded at startup; at expiry, does it update automatically, warn in advance, or fail suddenly |
| State drift | After running for a long time, the in-memory state slowly drifts away from the real external state (route tables, files, remote config, name resolution results); is there periodic reconciliation |
| Periodic maintenance tasks | Do tasks such as compaction, cleanup, rotation and renewal stop running for good after failing once; is the failure noticed |
| **Sleep and wake** | After the host sleeps, the process is suspended or the container is frozen and then resumes: timers fire all at once as a spike; leases and connections are no longer valid but are still treated as valid locally; does the monotonic clock keep counting during sleep (check per platform) |
| Long-duration verification | Do components that claim to be resident have a way to be verified by running for a long time or with accelerated time; running for only a few minutes does not prove long-term steady state |
