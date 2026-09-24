# 18 Concurrency and memory model

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
| Reentrancy | Can the same execution flow take a non-reentrant lock twice; can callback, notification or cleanup paths come back into the same operation while it is still running | Self-deadlock; on reentry the state is only half changed, causing a repeated close or unbounded recursion |
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
| Clock coarseness | Places that use timestamps to decide order — see [17](17-time-and-clocks.md) | Two writes in the same tick cannot be told apart |
| Test synchronization hooks | Wait/idle flags in production code that exist only for tests | Record and explain them; move them into test files if possible |
| Deterministic reproduction | Can a deterministic interleaving be built to reproduce it, instead of running it again and again and hoping to hit it | Only luck → even after a fix, nothing proves it |
