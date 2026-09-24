# 40 Testability and fault injection

> This item does not review "are the tests well written" but "**is there any way to test this code**".

| Checkpoint | What counts as a problem |
| --- | --- |
| Dependency injection points | Clock, file system, network, random numbers and external services are hard-coded as globals, so behavior cannot be controlled |
| Fault injection | Storage errors, short writes, dropped connections, partial failures, out of space and dependency timeouts cannot be produced → error paths can never be tested |
| Deterministic interleaving | Concurrency issues can only be hit by running again and again and hoping; there is no deterministic way to reproduce them |
| Observable results | The claimed behavior cannot be verified through return values, state or side effects; when public results can verify it, do not call it an issue just because private fields cannot be seen |
| Environment dependencies | It cannot be tested without a real database / network / privileges |
| **Cost constraints** | Prefer building branches from existing inputs and dependencies; whether internal changes are allowed follows the project rules; do not add unrelated abstractions, public switches or hot-path cost for tests; when it cannot be verified reliably, state the boundary |
