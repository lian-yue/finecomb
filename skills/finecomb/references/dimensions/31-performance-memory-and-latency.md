# 31 Performance, memory and latency

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
| Artifact size | How do the size and dependency count of artifacts, images and packages change with the change; was a whole set of dependencies pulled in for one small feature |
| **Optimized paths that skip validation** | Do fast paths, caches, batch verification and incremental computation added for performance skip the security or correctness checks on the slow path; are there tests comparing the fast and slow paths |

**Discipline for reporting performance conclusions**: any cost you give is either measured, or has the basis of its estimate written out (how many system calls, how many atomic operations, the order of magnitude against a baseline). **Never present a number you did not measure as a measured one**.
