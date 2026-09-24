# A.7 C# and .NET

| Checkpoint | What counts as a problem |
| --- | --- |
| async void | `async void` outside event handlers: an exception may terminate the process directly, and callers cannot await it |
| Waiting synchronously on async code | `.Result` and `.Wait()` deadlock in environments that have a synchronization context; library code without `ConfigureAwait(false)`; nobody observes the exceptions of tasks that are not awaited |
| Resources | `IDisposable` not released with `using`; creating a new `HttpClient` per request exhausts ports |
| **Deserialization** | `BinaryFormatter` (disabled or removed in newer versions) and Json.NET with `TypeNameHandling` set to anything other than `None` execute gadget chains |
| String comparison | `StartsWith(string)`, `string.Compare` and similar compare using the current culture by default; identifiers and security decisions need `StringComparison.Ordinal` |
| Integers | Overflow is not checked by default (except in a `checked` context) |
| Value types | A `struct` is copied by value; changing a copy of a mutable `struct` taken from a collection has no effect |
| LINQ deferred execution | Enumerating more than once runs the query again; variables captured by closures are changed before the query runs |
| Time | Mixing `DateTime.Kind` values; mixing `DateTime.Now` and `DateTime.UtcNow` |
| Nullable reference types | They are only compile-time warnings and are not checked at run time |
| Regular expressions | No timeout by default, so backtracking can be exploited; set a timeout, or use `NonBacktracking` (available from .NET 7) |
| Lock objects | `lock(this)`, or locking on a type object or a string: outside code may lock the same object |
