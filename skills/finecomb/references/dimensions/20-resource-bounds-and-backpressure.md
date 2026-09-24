# 20 Resource bounds and backpressure

| Checkpoint | What counts as a problem |
| --- | --- |
| Can the bound be computed | Is the worst-case use of memory, CPU time, execution flows, handles and connections effectively bounded; when this cannot be determined, record the missing conditions, and report it as unbounded only when continued growth is proven |
| Unbounded structures | Queues, buffers, maps, arrays and waiter lists that only grow and never shrink, with no eviction or limit |
| Amplification factor | How many internal operations, allocations and system calls one external request turns into; whose input decides the amplification factor |
| **Cost asymmetry** | The peer does something cheap (starts and then cancels right away, resets a stream, leaves a connection half open, submits and then withdraws), while this side has to finish expensive work; do cancelled operations still hold resources outside the concurrency limit |
| Slow consumers | What happens when production is faster than consumption |
| Backpressure policy | Once full, does it block, drop or return an error — the behavior is not defined; or the chosen policy lets one slow party hold resources that everyone needs (a shared lock, buffer, connection or worker) |
| Recursion depth | Is the depth bounded; can external input blow the stack |
| Batch entry points | Do batch entry points that accept arguments of any length have a limit; what happens on one huge call |
| **Hash collision flooding** | Untrusted strings used as hash table / map keys, where collisions let a single request use up the CPU; does every key form go through the seeded hash, including integer-like strings and keys whose hash is cached or precomputed |
| Algorithmic cost | Can external input trigger regex backtracking, combinatorial search or other superlinear computation; judge by the actual algorithm and engine, and do not report denial of service based only on the shape of an expression. How to judge a backtracking regex: adjacent or nested quantifiers can match the same stretch of characters; when a library builds the regex from a template, check the generated result for every template shape |
| **Work repeated across incremental input** | In streaming or incremental processing, input already examined is scanned again from the start each time more arrives; each call looks linear, but the total grows with the square of the input |
| **Limits that cover every branch** | Limits on size, count, time or concurrency are attached to only some branches (for example they apply only to network streams, while the branches taken by inline data, local files, or another protocol or encoding are not limited) |
| **Work after a limit is exceeded** | When a size, count or time limit trips, processing does not stop at once (reject, reset, close); the code keeps reading, decoding or discarding the rest (for example to keep protocol state in sync), and that discard path has no bound in bytes, frames or CPU |
| Concurrency limit | Is there a limit on operations in flight at the same time; who controls it; can slow or idle peers fill the limit cheaply, and does a full limit starve everyone else |
| Quota ownership and accounting | Are limits bound to a trusted account or tenant; can switching addresses, batching requests or switching nodes bypass them; is budget reserved before expensive operations, and settled correctly after cancellation and failure |
| **Storage of limit state** | Counters, lockouts and quotas are kept where the limited party can make them disappear: a bounded cache it can flush by using other keys, a short expiry, memory lost on crash or restart, or a copy per replica so another node starts from zero |
