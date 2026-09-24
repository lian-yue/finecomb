# 42 Coverage

Analyze uncovered behavior in scope only when there is a valid coverage record, or collecting one has been approved for this run. Coverage only describes what was executed; it cannot prove that assertions are effective or that the code is secure. When there is no record, say so plainly, and do not generate a full baseline on your own.

| Category | Handling |
| --- | --- |
| Testable but not tested | A read-only review reports the behavior gap; when changes are authorized, add tests for the behavior this change affects, per the project test strategy |
| Unreachable | Handle it under [dead code](1-dead-code-and-reachability.md); do not write tests for it |
| Reachable only with injected faults or on specific platforms | Record it as an unverified boundary and give the reason; for whether faults can be injected, see [40](40-testability-and-fault-injection.md) |

Give priority to real success, rejection and failure semantics; for the cost constraints on implementing tests, see [40](40-testability-and-fault-injection.md).
