# A.7 C#, F# and .NET

| Checkpoint | What counts as a problem |
| --- | --- |
| async void | `async void` outside event handlers: an exception may terminate the process directly, and callers cannot await it |
| Waiting synchronously on async code | `.Result` or `.Wait()` blocks a context needed by the awaited continuation; required context is lost through inappropriate continuation settings; nobody observes the exceptions of tasks that are not awaited. The absence of `ConfigureAwait(false)` alone is not a defect |
| Resources | Owned `IDisposable` resources are not released on every required path; repeated creation of independent HTTP connection pools exhausts ports; pooled or factory-managed handlers are mistaken for per-request connections |
| **Deserialization** | `BinaryFormatter` (disabled or removed in newer versions) and Json.NET with `TypeNameHandling` set to anything other than `None` execute gadget chains |
| String comparison | The selected comparison overload uses culture or case rules that disagree with the identifier or protocol contract; choose the required ordinal or culture-aware semantics explicitly |
| Integers | Overflow is not checked by default (except in a `checked` context) |
| Value types | A `struct` is copied by value; changing a copy of a mutable `struct` taken from a collection has no effect |
| LINQ deferred execution | Enumerating more than once runs the query again; variables captured by closures are changed before the query runs |
| Time | Mixing `DateTime.Kind` values; mixing `DateTime.Now` and `DateTime.UtcNow` |
| Nullable reference types | They are only compile-time warnings and are not checked at run time |
| Regular expressions | The actual pattern, engine and reachable inputs permit work beyond the accepted budget; assess configured timeouts, limits and engine semantics using [A.32](lang-regex.md), rather than treating the absence of a timeout alone as proof of denial of service |
| Lock objects | `lock(this)`, or locking on a type object or a string: outside code may lock the same object |
| Random numbers | `System.Random` is not a cryptographic random source; use `RandomNumberGenerator` |
| Async disposal and borrowed memory | Cancellation skips required `IAsyncDisposable` cleanup; pooled buffers are returned while asynchronous users still hold them; a `ValueTask` is consumed more times than its source permits |
| F# workflows and sequences | An `Async` workflow is assumed to have started when only constructed; repeated starts or sequence enumeration repeat effects; lazy enumeration escapes a resource's `use` scope |
| Trimming and AOT | Reflection, dynamic code or serialization relies on metadata or members removed by the deployed build; the development runtime does not exercise that path |

F# semantic reference: [async and task programming](https://learn.microsoft.com/en-us/dotnet/fsharp/tutorials/async).
