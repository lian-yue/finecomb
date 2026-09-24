# Part II: Five question lists (the main way to find real issues)

This part applies the checks to concrete objects; [Part III](dimensions/index.md) and [Part IV](specialties/index.md) hold the detailed criteria. When one object matches several places, reuse the same evidence and conclusion. Do not run the check again or report it twice.

## 2.1 For each public entry point

| Question | Matching criteria |
| --- | --- |
| Who can call it, which parameters can they control, and can prerequisite steps be bypassed? | [9 business flow](dimensions/9-business-logic-and-flow-integrity.md), [27 trust boundaries](dimensions/27-security-and-trust-boundaries.md), [28 authorization](dimensions/28-authorization-and-access-control.md) |
| What happens when each parameter or the receiver is null, extreme or invalid? | [10 API](dimensions/10-apis-and-contracts.md), [14 boundaries](dimensions/14-boundaries-numbers-and-text.md), [16 runtime](dimensions/16-language-and-runtime-pitfalls.md) |
| Who owns the returned resources and references, and can the caller modify or reuse them? | [10 API](dimensions/10-apis-and-contracts.md), [19 lifecycle](dimensions/19-lifecycle-and-resources.md) |
| What does a failure, exception or process termination at each step leave behind? | [13 errors](dimensions/13-error-handling-and-failure-semantics.md), [19 resources](dimensions/19-lifecycle-and-resources.md), [25 recovery](dimensions/25-crash-and-recovery.md) |
| Which state and copies does it modify, and which of them must take effect together? | [5 coupling](dimensions/5-coupling-and-blast-radius.md), [23 consistency](dimensions/23-transactions-atomicity-and-consistency.md) |
| What happens with the concurrency, callback reentry, close and repeated calls that the contract allows? | [12 state machine](dimensions/12-state-machines-and-transitions.md), [18 concurrency](dimensions/18-concurrency-and-memory-model.md), [23 idempotency](dimensions/23-transactions-atomicity-and-consistency.md) |
| Can cancellation, a timeout, or a slow or failing dependency stop the whole operation? | [17 time](dimensions/17-time-and-clocks.md), [22 degradation](dimensions/22-fault-isolation-and-degradation.md), [4.1 connections](specialties/4.1-networking-and-connections.md) |
| Which external operations or interpreters does the input reach, and who decides the cost? | [20 resources](dimensions/20-resource-bounds-and-backpressure.md), [27 trust boundaries](dimensions/27-security-and-trust-boundaries.md) and the matching specialties |
| Which obligations is the caller likely to miss, and do the docs match the implementation? | [11 misuse resistance](dimensions/11-usability-and-misuse-resistance.md), [43 documentation](dimensions/43-documentation-consistency.md) |

## 2.2 For each piece of shared state (fields, globals, caches, counters, connections, files)

| Question | Matching criteria |
| --- | --- |
| Who owns, publishes and modifies it, and does the protection cover compound operations? | [18 concurrency](dimensions/18-concurrency-and-memory-model.md) |
| How many copies exist across data, counters, indexes, caches and remote ends, and who brings them in line, and when? | [5 coupling](dimensions/5-coupling-and-blast-radius.md), [23 consistency](dimensions/23-transactions-atomicity-and-consistency.md) |
| Are growth and shrinkage bounded, and does it block scaling out to multiple instances? | [20 resources](dimensions/20-resource-bounds-and-backpressure.md), [21 capacity](dimensions/21-scalability-and-capacity.md) |
| Who cleans up after close and after a crash, and how is a trusted state restored? | [19 lifecycle](dimensions/19-lifecycle-and-resources.md), [25 recovery](dimensions/25-crash-and-recovery.md) |
| What sensitive data does it hold, who can access it, and when does it expire or get deleted? | [28 authorization](dimensions/28-authorization-and-access-control.md), [30 privacy](dimensions/30-privacy-data-governance-and-compliance.md), [4.4 credentials](specialties/4.4-cryptography-and-credentials.md) |

## 2.3 For each invariant (written in flow documents or claimed in code comments)

| Question | Matching criteria |
| --- | --- |
| Who guarantees it, and do all reachable entry points and failure paths maintain it? | [8 correctness](dimensions/8-functional-correctness-and-fitness-for-requirements.md), [12 state machine](dimensions/12-state-machines-and-transitions.md), [23 transactions](dimensions/23-transactions-atomicity-and-consistency.md) |
| Does it still hold with multiple processes or nodes, or when recovery itself fails again? | [25 recovery](dimensions/25-crash-and-recovery.md), [4.25 distributed coordination](specialties/4.25-distributed-coordination-and-leases.md) |
| What is the impact when it breaks, when is that detected, and what restores it? | [22 degradation](dimensions/22-fault-isolation-and-degradation.md), [33 observability](dimensions/33-observability-and-diagnosability.md) |
| Do code and documents agree, and can you build a counterexample? | [43 documentation](dimensions/43-documentation-consistency.md), [V evidence](report.md#part-v-evidence-levels-and-report-format) |

## 2.4 For each external side effect (files, directories, locks, connections, network, environment, registry, databases, queues, external services)

| Question | Matching criteria |
| --- | --- |
| Are the target and the operation authorized, and can input widen the scope of impact? | [28 authorization](dimensions/28-authorization-and-access-control.md), [29 dangerous operations](dimensions/29-fail-safe-behavior-and-dangerous-operations.md) and the matching specialties |
| Are "write finished", "durably committed" and "success returned" the same moment? | [23 transactions](dimensions/23-transactions-atomicity-and-consistency.md), [25 recovery](dimensions/25-crash-and-recovery.md) |
| After a partial write, a process crash or a repeated open, how are leftovers detected and handled? | [24 formats](dimensions/24-encoding-and-persistent-formats.md), [25 recovery](dimensions/25-crash-and-recovery.md) |
| Who is responsible for releasing it, and what happens on resource exhaustion, mount changes and permission failures? | [19 lifecycle](dimensions/19-lifecycle-and-resources.md), [34 environment](dimensions/34-runtime-environment-and-deployment-contract.md) |
| Do peer timeouts, lost responses and retries cause duplicates or amplification? | [22 degradation](dimensions/22-fault-isolation-and-degradation.md), [23 idempotency](dimensions/23-transactions-atomicity-and-consistency.md) |
| When resources are shared with other users, processes or instances, do they interfere with each other? | [37 coexistence](dimensions/37-interoperability-and-coexistence.md), [4.8 paths](specialties/4.8-file-systems-and-paths.md) |
| Who restores the system-level changes left after a crash or a forced kill (routes, name resolution settings, firewall rules, mounts, registry)? | [25 recovery](dimensions/25-crash-and-recovery.md), [4.28 data plane](specialties/4.28-tunnels-proxies-and-the-network-data-plane.md), [4.15 process entry points](specialties/4.15-command-line-and-process-entry-points.md) |

## 2.5 For each background flow and automatic behavior (threads, coroutines, timers, finalizers, code run at import or load, signal handlers, listener callbacks)

| Question | Matching criteria |
| --- | --- |
| Who starts it, what triggers it (startup, timer, event, import, reclamation), and can external input trigger it or speed it up? | [12 state machine](dimensions/12-state-machines-and-transitions.md), [20 resources](dimensions/20-resource-bounds-and-backpressure.md), [4.11 scheduling](specialties/4.11-timers-and-scheduling.md) |
| Which shared state does it read and write, and does the protection still cover it when it runs concurrently with public entry points? | [18 concurrency](dimensions/18-concurrency-and-memory-model.md) |
| When it errors, crashes or hangs internally, who finds out, and can it take down the process or silently stall? | [13 errors](dimensions/13-error-handling-and-failure-semantics.md), [22 degradation](dimensions/22-fault-isolation-and-degradation.md), [33 observability](dimensions/33-observability-and-diagnosability.md) |
| Does it back off on repeated failures, or can it turn into a hot loop or a log flood? | [22 degradation](dimensions/22-fault-isolation-and-degradation.md), [45 long-running](dimensions/45-long-running-and-resident-processes.md) |
| On close, who stops it, does anyone wait for it, and after it stops can it still change state or hold resources? | [19 lifecycle](dimensions/19-lifecycle-and-resources.md), [12 state machine](dimensions/12-state-machines-and-transitions.md) |
| Does it still wake up, poll or scan while idle, and after running for a long time are counters and containers still bounded? | [45 long-running](dimensions/45-long-running-and-resident-processes.md), [20 resources](dimensions/20-resource-bounds-and-backpressure.md) |
| With multiple instances, after a restart, or after the host sleeps and wakes, can it run twice, miss a run, or have instances compete with each other? | [4.11 scheduling](specialties/4.11-timers-and-scheduling.md), [4.25 distributed coordination](specialties/4.25-distributed-coordination-and-leases.md), [34 environment](dimensions/34-runtime-environment-and-deployment-contract.md) |
| Does the flow document state its triggers, results and how it stops? | [43 documentation](dimensions/43-documentation-consistency.md) |
