# 19 Lifecycle and resources

| Checkpoint | What counts as a problem |
| --- | --- |
| Acquire/release pairing | Created but never released; **early returns on error paths skip the release** |
| Placement and order of deferred release | Registered too late; the execution order of several deferred calls is the reverse of the release order that is needed |
| Manual release | Release written by hand in several places in the same function (miss one and it leaks) — use automatic release where possible |
| **Internal order of close** | When closing, is there a reason for the release order of multiple resources (stop background work before closing handles? wake waiters before clearing state?), and is it written down |
| **Whether close waits** | Does it wait for in-flight operations? For how long? Are the consequences of not waiting (half-finished results, dirty snapshots) written down clearly |
| Reference counting | Increments and decrements do not pair up; a decrement is missed on an error path |
| Objects invalidated within a batch | Within the same batch, transaction or packet, can a later step still reach an object that an earlier step deleted, freed or invalidated |
| **Scope of error handling in a batch** | Code handling one item of a list, chain or batch handles an error with a helper that acts on every item (sends errors for, marks, resets, drops or frees all of them), so items it never examined are changed while other steps still process them |
| Allocation ownership | Is an object allocated from a shorter-lived memory pool, arena or scope attached to a longer-lived object, so that the longer-lived one still holds the pointer after the shorter-lived one is freed |
| **Finishing twice** | Can operations such as close, finish, release and dequeue run on the same object once from each of two paths; are registering, enqueuing and delivering idempotent (what happens when something already in the queue is registered again); does the second run decrement a count again or release again |
| Allocation in cleanup paths | Creating objects, allocating resources or sending notifications during close or cleanup; can that trigger collection, timeouts or callbacks that come back and close the object already being closed, causing a double release or unbounded recursion |
| Repeated close | The behavior does not match the contract, or it releases again a resource already handed over to another object |
| After close | Is the behavior of each entry point defined and consistent — see [12](12-state-machines-and-transitions.md) |
| Handles / connections / finalizers | Are there leak paths (for timers see [17](17-time-and-clocks.md)) |
| Temporary files and directories | Left behind on failure paths |

> Unbounded growth and whether the bound can be computed are checked in [20 Resource bounds and backpressure](20-resource-bounds-and-backpressure.md); memory leak patterns are checked in [31 Performance, memory and latency](31-performance-memory-and-latency.md).
