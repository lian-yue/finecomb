# 41 Test quality

| Checkpoint | How to check | What counts as a problem |
| --- | --- | --- |
| Always-true assertions | Static analysis + manual reading | Checks that can never fail |
| Tests of implementation details | Read what the assertions target | Only the internal implementation is verified, not the external behavior |
| Redundancy | Compare the behavior the cases cover | Several cases verify the same branch of the same behavior |
| Unused helpers | Static analysis | Test helper functions that nobody calls |
| Assertion precision | Read the assertions | Only checks "there is an error", not which error; only checks "there is a value", not whether the value is right |
| Flakes | First read the existing failure evidence and synchronization conditions; any runs needed follow the project test strategy | Intermittent failures are hidden by reruns, or the failure conditions are not recorded |
| **Can the test really catch the bug** | Compare against the triggering input, the key assertions and the evidence from before the fix; for isolated comparison see [Execution boundaries during review](../scope.md#execution-boundaries-during-review) | The case passes, but it does not reach the issue path, or its assertions do not check the actual effect |
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
| Race detection | Check the paths, interleavings and tool records of the concurrency verification | Interleavings that never ran, or logical atomicity, treated as proven; for the limits of the conclusion see [18](18-concurrency-and-memory-model.md) |
| Negative cases | Look for assertions that "what should not happen really did not happen" | Only the success path is tested |
| Side effects after rejection | Check the failure assertions for unauthorized access, invalid signatures, illegal input and the like | Only the error code is checked, with no check that data was not written, quota was not deducted, external calls were not made or sensitive fields were not returned |
| **Regression protection for security fixes** | Check against vulnerabilities and security issues fixed in the past | A fixed issue has no matching regression test, so nothing fails when a refactor deletes or bypasses the fix logic; the fix only blocks the sample input from that time |
| **Tests from the attacker's side** | Check against validators, checkers, constraints and permission checks | Tests only use honest callers and correct data; there are no cases like "a forged proof or signature, one tampered field, or one swapped binding value must be rejected"; deleting any single check or constraint still leaves all tests green |

For the maintenance scope of tests, see [39](39-generated-artifacts-and-toolchain.md).
