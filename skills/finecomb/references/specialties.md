# Part IV: Specialties by target type

The sections below add checkpoints for specific kinds of targets. Choose them based on the [factual baseline](baseline.md). General criteria are reused through links, and the check status is recorded only once, in the [report](report.md#part-v-evidence-levels-and-report-format).

## 4.1 Networking and connections

Applies to: any code that opens, holds or reuses connections: clients, servers, proxies, name resolution and transport-layer wrappers.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Timeout layers** | Do connect, handshake, read/write, idle and overall waits match the contract? Can an attacker hold resources that cannot be cancelled, reclaimed or put under backpressure? Do not call it a defect only because a long-lived connection has no overall time limit |
| Deadline propagation | Is the upstream's remaining time limit turned into read/write deadlines on the connection? After cancellation, does the connection really close, or does only the function return? |
| Connection ownership | Who is responsible for closing it? Is it closed on error paths? Who owns it after it is returned to the caller? |
| Half-close | Semantics of closing only the write direction; after the peer half-closes, can this end still read? |
| Connection leaks | Is the old connection closed on retry? Does the return path for pooled connections cover every exit? |
| Connection pools | Upper limit, idle reclamation, health checks, the usability check before reuse; when the pool is full, does it block, open a new connection or return an error? |
| Reconnect and backoff | Is there backoff? Does the backoff have a cap and **jitter**? How does the retry count relate to whether the operation is idempotent? |
| **Telling end of stream from timeout** | Are normal peer close / read timeout / network error / connection reset told apart and handled separately? Lumping them together makes a normal close get retried as a failure |
| Short reads and writes | Treating a partial read as a complete one; the byte count returned by a write is not checked; a single read is used where the buffer must be filled |
| Address families | Dual stack, concurrent attempts and fallback, behavior when one family is unavailable |
| Name resolution | Resolution timeouts, negative-result caching, where the TTL comes from, how it relates to the system resolution path; order and rotation of resolved results |
| Self-implemented name resolution | When you implement your own resolver or name service: are the transaction ID and source port random? Does the answer match the question section? Is the authority scope of the answer checked (against poisoning)? After truncation, does it switch to a reliable transport? Length of alias chains and loops in them; clamping TTLs to lower and upper bounds |
| Transport layer security | Minimum protocol version, **whether certificate verification has been turned off**, server name indication, application-layer protocol negotiation, session resumption, root certificate updates, certificate expiry alerts |
| Multiple hops and proxy chains | Error attribution, stacked timeouts, passing of credentials, fallback on failure |
| Address reuse and binding | Address/port reuse options, handling of bind failures, port exhaustion |
| Keepalive and hung peers | Transport-layer keepalive and application-layer heartbeats; how long does it take to notice a hung peer? |
| Backpressure | What happens to the writing side when reading is slow? Is the send buffer unbounded? See [20](dimensions.md#20-resource-bounds-and-backpressure) |
| Concurrent use | Can the same connection be read and written concurrently? Is that in the contract? |
| External service contracts | Rate limits and quotas, retry budgets, idempotency keys, and whether the other side's error semantics are read correctly |
| **Outbound destination** | The destination address comes from external data, but only timeouts and certificates were checked — see [4.21](#421-outbound-requests-and-server-side-request-forgery) |

## 4.2 Protocols and frame parsing

Applies to: any code that parses bytes off the wire itself and keeps protocol state.

| Checkpoint | What counts as a problem |
| --- | --- |
| Message boundaries | Several messages in one read / one message split across reads; what happens when one read returns half a frame, or two and a half frames? |
| **Coexisting with another parser** | Your code and the upstream / downstream disagree on length, chunking or line endings → request smuggling — see [4.14](#414-server-request-handling-and-middleware) |
| **Maximum frame length** | Allocation is driven by a length field declared by the peer, with no cap → the remote end can exhaust memory |
| Invalid input | Invalid frames, invalid state transitions, reserved bits, unknown types: does it disconnect, skip or crash? |
| State machine | Are protocol state transitions complete? See [12](dimensions.md#12-state-machines-and-transitions) |
| Compression and extensions | Is decompression amplification (small input that decompresses into huge output) capped? |
| Encoding | Text encoding validation, invalid code points, mask handling |
| Fragmentation and reordering | Caps and timeouts for fragment reassembly; handling of out-of-order / duplicate pieces |
| Control messages | Priority and size limits of control messages such as heartbeats and close |
| Version negotiation | Handling of incompatible versions; can an attacker force a downgrade during negotiation? |

## 4.3 Decoding untrusted external data

Applies to: any code that consumes external files, external responses or user uploads.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Allocation driven by length fields** | Allocating directly by a length found in the input, when that length comes from an untrusted source → memory amplification |
| Decompression/decoding amplification | Are total output bytes, entry count, nesting depth and cumulative work bounded? Can a per-item limit be bypassed by nested archives or streaming input? |
| Timeout | Does decoding have an overall time limit? |
| Format confusion | Deciding the format by file extension or declared type instead of by content |
| External entities and references | Will the parser fetch external references (local files, remote addresses)? For outbound destinations see [4.21](#421-outbound-requests-and-server-side-request-forgery) |
| Partial decoding | Side effects already produced when decoding fails |
| Object merging and prototype pollution | Can special properties of an external object, through merging, assignment by path or copying, modify a shared prototype, class or global object (such as the JavaScript prototype chain, or the Python attribute chain through `__class__` and `__init__.__globals__`), and then affect permissions, configuration or later requests? |
| Multiple formats and duplicate fields | When the same bytes can be read as different objects by different parsers, do validation and use apply the same format, field precedence and handling of invalid input? |
| Decoders you depend on | Known vulnerabilities of third-party decoding libraries; are their dangerous features disabled? |
| **Decoding becomes execution** | Deserialization runs code; the decoder reads local files or reaches the network — see [4.22](#422-subprocesses-dynamic-execution-and-decoder-side-effects) |

## 4.4 Cryptography and credentials

Applies to: any code that touches keys, tokens, signatures or passwords.

| Checkpoint | What counts as a problem |
| --- | --- |
| Algorithm choice | Deprecated/weak algorithms are used; the mode of operation is chosen badly |
| **Randomness source** | Do keys and unpredictable tokens use a cryptographic random source? Is a failure of the random source ignored? |
| Nonces and initialization vectors | Do they meet the uniqueness or unpredictability requirements of the algorithm used? Can concurrency, restarts, restoring backups or counter wraparound cause reuse? |
| Authentication | Encryption without authentication (no message authentication code / no authenticated encryption) |
| Hashing and integrity | Is a non-cryptographic hash wrongly used for security integrity checks? A non-secret checksum cannot replace message authentication or a signature |
| Signature binding | Does the signature cover the fields, operation, tenant and protocol purpose that are actually used? Do canonicalization ambiguity, cross-purpose reuse or unsigned fields change the meaning of what gets executed? |
| Key and purpose separation | The same key is used for several purposes or for both directions of communication; the handshake does not bind the full negotiation transcript, so it can be downgraded or spliced; key derivation lacks a purpose label |
| Constant time | Keys, authentication codes or tokens are compared with a short-circuiting comparison |
| Key lifecycle | Do key generation, storage, use, rotation, revocation and destruction form a closed loop? Are the acceptance period for old keys and the behavior on rotation failure clearly defined? |
| Credential exposure | Do source code, configuration, samples, logs, errors, image layers, cache keys or crash dumps contain live secrets? When one is found, record only the redacted location and the impact, and do not spread the original value; deleting a copy does not mean the credential has been revoked |
| Derivation | A password is used directly as a key (no key derivation); derivation parameters are too weak |
| Error messages | Errors are too specific and leak "which step failed" |
| Serialization | Key material is exported without authorization, or goes in plain text into unprotected storage, transport or object representations |
| Certificates and trust chains | Is validation complete (validity period, chain, revocation, usage)? Is it skipped anywhere? |

## 4.5 Authentication, sessions and tokens

Applies to: any code that verifies identity, keeps login state, or issues or validates tokens.

| Checkpoint | What counts as a problem |
| --- | --- |
| Credential strength | Password policy; blocking of common passwords; **is the maximum length set too low?** Is pasting blocked? |
| Credential storage | Are the password hashing algorithm and its parameters strong enough? Is there a salt? Can the parameters be upgraded? |
| Multi-factor | Is there any? Bypass paths (remember this device, recovery flow, backup codes) |
| **Brute-force protection** | Login attempt limits and lockout policy; can the lockout itself be used as an attack (locking someone else out by spamming their account)? |
| Account enumeration | Can the responses of login / recovery / sign-up (wording, status, timing) reveal whether an account exists? |
| Account recovery | Is the recovery flow weaker than login? Validity period and single use of recovery tokens |
| Sensitive identity changes | When changing the password, email or multi-factor settings, or linking an external identity, is current control of the account verified? After recovery, do old sessions and old recovery credentials become invalid as the contract says? |
| External identity binding | Is the account located by a trusted issuer and a stable subject identifier? Do unverified emails or accounts with the same name lead to automatic merging, wrong binding or takeover? |
| Session lifecycle | Creation, renewal, expiry, invalidation; are there both an idle timeout and an absolute timeout? |
| Session fixation | Is a new session identifier issued after successful authentication? |
| Concurrent sessions | How many are allowed? Does logging out in one place affect the others? Can active sessions be seen? |
| Token scope | Is the permission scope minimal? Can the token be used for operations or audiences beyond what was intended? |
| **Revocation** | Is there a revocation mechanism? How long until a revocation takes effect? How are stateless tokens revoked? |
| Refresh tokens | Are they rotated? Can theft be detected (replay detection)? |
| Storage and transport | Where are they stored and how are they sent? Do they show up in URLs, logs, storage readable by the front end, or caches? |
| Cookie boundaries | Do Secure, HttpOnly, SameSite, Domain and Path fit the session's purpose? Do same-name cookies, writes from subdomains and the deletion scope on logout change which session is actually selected? |
| Binding | Is the token bound to the client, device or address? What are the consequences of not binding it? |
| Validation completeness | Is the signature algorithm decided by the input? Are audience, issuer, validity period and purpose all validated? |
| Verification key selection | Are identifiers such as `kid` looked up only in a trusted key set? Is a key, certificate or remote key URL carried by the token trusted directly, or used for arbitrary file and network access? |
| Delegated authorization protocols | When an authorization framework is used: redirect URI allowlist, parameters against cross-site requests and replay, authorization code protection, token audience and scope, consent scope |
| **Redirect after login** | The return address after successful authentication comes from a request parameter and is not restricted → the token is sent somewhere else — see [4.18](#418-user-interfaces-and-accessibility) |
| Clock skew | How much clock skew does the expiry check tolerate? How large is the tolerance window? |

## 4.6 Caching and storage

Applies to: any code that stores data and reads it back. The general dimensions already cover most of this ([23](dimensions.md#23-transactions-atomicity-and-consistency) [24](dimensions.md#24-encoding-and-persistent-formats) [25](dimensions.md#25-crash-and-recovery) [26](dimensions.md#26-release-upgrade-migration-and-rollback) [20](dimensions.md#20-resource-bounds-and-backpressure)); only the specific points are listed here.

| Checkpoint | What counts as a problem |
| --- | --- |
| Eviction policy | Does the implementation really follow the promised eviction order? Do reads change the eviction order? |
| Penetration, stampede and avalanche (misses for keys that do not exist, a hot key expiring, many keys expiring at once) | Do all misses go through to the downstream? Are concurrent misses on the same key merged? Should expiry times get jitter? |
| Invalidation propagation | How are entries invalidated across replicas / instances? Is there a consistency promise? See [23](dimensions.md#23-transactions-atomicity-and-consistency) |
| Hit rate observability | Is the hit rate exposed? Is it exact or sampled? |
| Warm-up and cold start | After a restart, how long until the hit rate is usable again? |
| Negative-result caching | Is "does not exist" cached? For how long? |
| Cache keys and isolation | Do keys include the tenant, principal, permissions or representation dimensions that decide the response? Do negative results, shared object pools or error responses leak into other requests? Raw credentials must not be used as keys |
| Atomic publish | Temporary file plus atomic replace, or direct writes? Is syncing to disk covered? See [25](dimensions.md#25-crash-and-recovery) |
| Reclamation and compaction | When garbage collection, compaction or expiry cleanup interleaves with concurrent reads, backfill or promotion, does it delete data that is still referenced or still being written? Can the state be recognized when reclamation stops midway? Is space really freed? |
| Deletion and resurrection | After a delete, can concurrent read backfill, replication, promotion or replay bring the deleted entry back? Are there tombstones or version comparisons to block it? |

## 4.7 Databases and queries

Applies to: any code that accesses a data store through a query language.

| Checkpoint | What counts as a problem |
| --- | --- |
| **N+1** | Querying one item at a time inside a loop |
| Index use | Can the query use an index? Is there a full table scan? Do the indexes match the actual query patterns? Are there indexes nobody uses? |
| **Pagination stability** | Offset pagination misses or repeats rows under concurrent writes; is a stable cursor used? |
| Result set size | Queries with no limit; pulling a whole table at once; streaming reads or loading everything into memory |
| Long transactions and locks | Slow work inside a transaction (network calls, heavy computation); scope and duration of locks; deadlocks and retries |
| Batching | Batch size; committing too much at once makes rollback expensive and bloats the log |
| Connection pool | Limit, timeouts, leaks, and fit with the concurrency level and the database's own limit |
| Query injection | Are data values parameterized? Do structures that cannot be parameterized, such as table names, column names and sort expressions, use an allowlist? Do ORM raw queries, NoSQL operators and search expressions still accept arbitrary input? |
| Data-layer permissions | Does the database account have more privileges than its job needs? With row-level policies or a connection-level tenant context, can a reused connection keep the identity of the previous request? |
| Migrations | Are they reversible? Do they lock tables? Strategy for adding columns / adding indexes / changing types on large tables; ordering relative to code releases |
| Nulls and defaults | Mapping between nulls in storage and nulls in the language; which layer holds default values |
| Time and time zones | Does the stored time type carry a time zone? Are the same time zones used for writing and reading? |
| Read/write splitting | Can reads from a replica return stale data? Can read-your-writes be guaranteed? |
| Dialects and implicit behavior | Three-valued logic of nulls, implicit type conversion, collation and trailing-space comparison, truncation in non-strict mode, and default isolation levels differ between databases; for details see [the SQL section of Appendix A](lang-sql.md) |

## 4.8 File systems and paths

Applies to: any code that works on files and directories directly.

| Checkpoint | What counts as a problem |
| --- | --- |
| Path construction | Is the object finally accessed still inside the allowed root? Can absolute paths, parent-directory steps, encoding differences or string-prefix-only comparison bypass the restriction? |
| Links and replacement after check | When a symbolic link, hard link, mount or parent directory is replaced after validation, do the actual reads, writes or deletes go out of bounds? Normalizing the path once does not guarantee the object stays the same afterwards |
| Atomic replace | Check against the actual system, file system and language API. Crossing directories does not necessarily lose atomicity, and staying in the same directory is not a cross-platform guarantee (POSIX `rename` and Windows replace semantics differ, and each language wraps them differently). Judge cross-file-system limits and durability across power loss separately; for example, [Go os.Rename](https://pkg.go.dev/os#Rename) states the platform differences |
| Temporary files | Is the name predictable (can someone take it first)? Permissions at creation; are they deleted on failure paths? |
| Directory iteration | Iteration order is not guaranteed; what happens when entries are added or removed during iteration? |
| Large directories | Cost of listing when there are many entries; are they sharded? |
| File locks | Advisory or mandatory locks; semantic differences across platforms; are locks released automatically after a process crash? |
| Space and quotas | Is space checked before writing? Cleanup after a failed write |
| Permissions and ownership | Permissions at creation; inherited ownership; effect of the permission mask |
| Path length and characters | Length limits, illegal characters, reserved names, case, normalization forms |
| **Uploads and foreign files** | Type and size validation; is the storage location executable? Is the file name rewritten? Content type and disposition headers on download |
| Archive extraction | Are the final path, link target and file type of each entry restricted? Can nested archives, duplicate names or link entries overwrite files outside the root or existing trusted files? |
| Upload publishing and download | Is the file accessible before validation and processing finish? Do chunk merging and resumed uploads check ownership? Are download links bound to the resource, operation and validity period? Can user content run in the context of a trusted site? |
| Watching and notification | Reliability of file change watching (lost, duplicated, batched events); is there a fallback poll? |

## 4.9 Events, publish-subscribe and observers

Applies to: any **in-process** code with callbacks, listeners, subscriptions or notifications. For cross-process delivery, see [4.10](#410-message-queues-and-async-jobs).

| Checkpoint | What counts as a problem |
| --- | --- |
| Subscription lifecycle | The path to unsubscribe; are events still received after unsubscribing? Double unsubscribe; leaks from forgetting to unsubscribe |
| **Slow subscribers** | Can one stuck subscriber hold up the publisher or other subscribers? Is the buffer unbounded? |
| Delivery semantics | At most once / at least once: is it promised, and does the implementation match? |
| Ordering | Is ordering promised? And across multiple subscribers? |
| Callback execution context | Does it run in the publisher's execution flow or in a separate one? **Are callbacks invoked while holding a lock?** (see [18](dimensions.md#18-concurrency-and-memory-model)) |
| Callback crashes | Can one subscriber crashing bring down the publisher? |
| Reentrancy | What happens when a callback subscribes / unsubscribes / publishes again? |
| Dropped events | When full, does it drop the oldest, drop the newest or block? Is a drop reported? |
| Shutdown | What happens to in-flight events at shutdown? How do subscribers find out? |

## 4.10 Message queues and async jobs

Applies to: any code that delivers messages across processes or submits background jobs.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Delivery semantics and idempotency** | Under at-least-once delivery, is the consumer idempotent? Where does the idempotency key come from? See [23](dimensions.md#23-transactions-atomicity-and-consistency) |
| Ordering | Is there an ordering guarantee? Ordered within a partition or globally? Can retries break the order? |
| Offset commit | Commit the offset first or process first? Where does it resume after a crash? Can messages be lost or duplicated? |
| **Dead letters and poison messages** | Where do messages that keep failing go? Is there a dead-letter queue with alerting? Can one bad message block all consumption? |
| Backlog | How is backlog observed? Degradation strategy under backlog; can consumers be scaled out? |
| Consumer groups and rebalancing | Duplicate consumption and interrupted processing during rebalancing |
| Message size and evolution | Size limit; can old consumers read new fields? See [4.16](#416-data-models-and-generated-contracts) |
| Graceful stop | What happens to messages being processed when stopping? Can they be lost? |
| Delay and retry queues | Precision of delayed messages; can the retry queue loop forever? |
| Transactional send | What if "local state changed but the message was not sent"? Is there an outbox pattern? |
| Message identity and authorization | Are the producer, topic, tenant and target resource trusted? Are current permissions checked on consumption and on manual replay, rather than trusting only the identity the message claims for itself? |

## 4.11 Timers and scheduling

Applies to: any code that runs on a time trigger.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Overlapping runs** | The next run fires before the previous one has finished; is overlap allowed? If not, what guarantees it? |
| Missed runs | Should runs missed during downtime be made up? How many times? Can catching up overwhelm the downstream? |
| **Single instance in a distributed setup** | Can several replicas run at the same time? What guarantees that only one runs? How long until the lock is released after its holder crashes? |
| Time zones and daylight saving | On the day of a time zone change / daylight saving switch, does the expression run twice or not at all? |
| Jitter | Thundering herd at the top of the hour; is jitter added? |
| Long jobs | What happens when a single run takes longer than the trigger period? |
| Failure and retry | Is a failed run made up? Can a retry collide with the next run? |
| Observability | Can the last run time, duration, result and next scheduled time be seen? |
| Manual trigger | Can it be triggered manually? Can manual and automatic runs happen concurrently? |
| Stop and drain | What happens to jobs that are running at shutdown? |

When distributed locks, leases or leader election are used, also check [4.25](#425-distributed-coordination-and-leases).

## 4.12 Dependency wiring and service lifecycle

Applies to: any code that assembles components and manages starting and stopping.

| Checkpoint | What counts as a problem |
| --- | --- |
| Start order | Is the dependency order explicit or accidental? Circular dependencies |
| Start failure | When startup fails halfway, how are the parts that already started cleaned up? |
| **Stop order** | Is it the reverse of the start order? Is there a timeout? After the timeout, does it force the stop or hang? |
| Readiness and health | Are "it is up" and "it can serve" the same thing? What does the health check actually check? |
| Singletons and scope | Who holds it, who releases it; the same dependency built twice |
| Background task ownership | Which component owns each background task, and who waits for it at stop? |
| Configuration injection | Is configuration validated at wiring time, or does it blow up only at run time? |
| Lazy initialization | Concurrency safety and failure retry of lazy loading |

## 4.13 Logs, metrics and tracing

Applies to: any code that produces observability data. Look here when it is **the object under review itself**; when the code is **a user** of it, see [33](dimensions.md#33-observability-and-diagnosability).

| Checkpoint | What counts as a problem |
| --- | --- |
| Allocation and hot paths | Does a call still allocate when its level is disabled? Cost of building labels |
| **Cardinality explosion** | Label values come from user input → the number of time series explodes |
| Concurrency | Is the output side safe for concurrent use? Do interleaved writes from many execution flows get serialized into a bottleneck? |
| Blocking | When the output target blocks, does it hold up business code? Is there a drop policy? |
| Sampling | Are sampling rules configurable, and are they biased? Is trace sampling consistent with log sampling? |
| Levels and switches | Can they be changed at run time? Does a change apply to existing instances? |
| Context propagation | Can trace identifiers be passed across execution flows and across processes? |
| Sensitive information | Is there a redaction hook? See [30](dimensions.md#30-privacy-data-governance-and-compliance) |

When messages go into an expression interpreter, see [4.22](#422-subprocesses-dynamic-execution-and-decoder-side-effects); when it records security events, see [4.23](#423-security-audit-logs) for integrity, retention and disposal requirements.

## 4.14 Server request handling and middleware

Applies to: any code that receives external requests and dispatches them for handling.

| Checkpoint | What counts as a problem |
| --- | --- |
| Middleware order | Is the order written down? Relative position of crash recovery, logging, authentication, authorization and rate limiting |
| Request body limit | No size limit |
| Timeouts | Do time limits, cancellation and backpressure at each stage fit the request type? Do handlers honor request cancellation? For long-lived connections, see [4.1](#41-networking-and-connections) |
| Graceful shutdown | Waiting and timeout at shutdown; in-flight requests and long-lived connections |
| Error mapping | How do internal errors become external status? Is internal information leaked? |
| Header handling | Injection, duplicates, case, forwarding of hop-by-hop fields, trust in the source address |
| **Host header** | When Host is used to build absolute URLs, reset emails or routes, can the caller control it? |
| **Cross-site request forgery** | When session credentials are attached to browser requests automatically, can a cross-site request trigger an entry point that changes state? Is relying on "same-site" alone enough? Is there a token or custom header? |
| Cross-origin access (CORS) | Are the allowed origins, methods, headers and credentials setting checked exactly? Do reflecting any origin, wrong subdomain matching or checking only preflight requests widen access? CORS cannot replace server-side authorization or CSRF protection |
| **Request smuggling and cache poisoning** | Do the proxy and the backend interpret length, chunking, protocol conversion, paths or headers the same way? Is an input that affects the response missing from the cache key, or is a sensitive response cached as a public static resource? |
| **WebSocket and long-lived connections** | Are origin and identity valid at the handshake? After that, does each message have permission checks and size limits? Do session expiry, revocation and tenant switching invalidate old connections? |
| Streaming responses | Flush timing, client cancellation, backpressure |
| Routing | Ambiguous path matching, trailing slashes, case, parent-directory steps |
| Proxy trust | Do headers carrying forwarded identity, source address, protocol or client certificate come only from trusted proxies? Do direct connections to the backend or changes in ingress routing bypass gateway checks? |
| Concurrency limits and rate limiting | Is concurrency capped for each handling path? What dimension is rate limiting keyed on? What is the response when over the limit? |
| Services listening only on the local machine | Binding only to the loopback address does not mean only trusted callers: web pages in a browser can reach it through DNS rebinding and cross-site requests, and other local users can connect too. Are Host, origin and caller identity validated? |
| Authorization placement | Are permission checks in middleware or in handlers? Can some path bypass them? See [28](dimensions.md#28-authorization-and-access-control) |

## 4.15 Command line and process entry points

Applies to: any code that serves as a process entry point.

| Checkpoint | What counts as a problem |
| --- | --- |
| Argument parsing | Required / default / mutually exclusive / validation; are error messages readable? |
| **Signal handling** | Handling of termination signals; forced exit on a second signal; reload signals |
| Graceful stop | Stop order, timeout, fallback after the timeout: do they match the shutdown contract in [34](dimensions.md#34-runtime-environment-and-deployment-contract)? |
| Exit codes | Are success / runtime failure / usage error told apart? The code on abnormal exit |
| Output streams | Is normal output kept apart from diagnostic output? Can it be read by machines? |
| Priority of configuration sources | See [32](dimensions.md#32-configuration-and-defaults) |
| Single instance | Is protection against starting twice needed? How are leftover process ID / lock files handled? |
| Privileges and environment | Is privilege elevation needed? The message when elevation fails; **can changed system state (network configuration, resolver settings, firewall) be restored after an abnormal exit?** See [29](dimensions.md#29-fail-safe-behavior-and-dangerous-operations) |
| Interactivity | In non-interactive environments (no terminal, pipes, services), does it hang waiting for input? |

## 4.16 Data models and generated contracts

Applies to: any code that defines data shapes across modules / processes / languages.

| Checkpoint | What counts as a problem |
| --- | --- |
| Field compatibility | Impact of adding / removing fields / changing types on upstream and downstream; are field identifiers reused? |
| Generation source | Was the definition changed, or the artifact? See [39](dimensions.md#39-generated-artifacts-and-toolchain) |
| Zero value and unset | Can the two be told apart after serialization? Semantics of default values |
| Storage mapping | Are the model and the table schema, indexes and migrations in sync? |
| Cross-language | Type mapping differences when code is generated for several languages (integer width, time, enums, nulls) |
| Enum evolution | How do old ends handle new enum values? Is there a fallback for unknown values? |
| Validation placement | Is structural validation in the model layer, or written again by every caller? |

## 4.17 Templates and text output

Applies to: any code that assembles text for a downstream interpreter: markup, queries, command lines, URLs.

| Checkpoint | What counts as a problem |
| --- | --- |
| Escaping | Producing markup with a template engine that does not escape; hand-building markup / queries / commands / URLs |
| Context-aware escaping | The same data needs different escaping in attributes, scripts, styles and URLs |
| User content | User-provided content is parsed as a template or as markup |
| Template source | Does the template itself come from an untrusted source? |
| Output size | Is the expanded output capped? |
| Line breaks and encoding | Injected line breaks break the structure (log injection, header injection); can terminal control characters fake the diagnostic display? Output encoding declaration |
| Spreadsheet export | When user data is exported to CSV or spreadsheets, can the reading software interpret it as formulas? Does the handling cover the software actually targeted, rather than relying only on the file extension? |

## 4.18 User interfaces and accessibility

Applies to: any code that produces a human-machine interface.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Accessibility** | Semantic structure, keyboard reachability and focus order, focus management, alternative text, contrast, motion that can be turned off, readability by assistive technology; association between form labels and error messages |
| State management | Is there a single source of state? Can derived state drift from its source? Syncing local state with remote state |
| Request races | A later request returns first and an older response overwrites the newer result; responses still write state after the view has changed |
| Duplicate submission | Can the button be clicked repeatedly? Is there idempotency protection? |
| The four states | Loading / empty / error / offline: are all of them designed? |
| Rendering performance | Unneeded re-renders; virtualization of long lists; long tasks on the main thread; layout thrashing |
| Asset size | Bundle size, loading on demand, size and format of images and fonts |
| Compatibility | Target browser / device range; fallback plan; feature detection rather than environment sniffing |
| Responsive layout | Breakpoints, extreme widths, zoom, landscape and portrait, enlarged fonts |
| **Client-side security** | Do DOM writes and dynamic execution accept unprocessed data? Are the content security policy, framing protection, and the origin, sender and structure checks on cross-window messages effective? For cookies see [4.5](#45-authentication-sessions-and-tokens); for CORS and CSRF see [4.14](#414-server-request-handling-and-middleware) |
| Browser persistent state | Do local storage, offline caches and Service Workers keep data from the previous account? Can it still be read after logout, tenant switching or revoked authorization? |
| **Open redirect** | Can external input make a redirect go beyond the allowed range, disguise itself as a trusted destination or carry credentials out? A redirect back after login must also meet the authentication binding in [4.5](#45-authentication-sessions-and-tokens) |
| Third-party scripts | Do included external scripts have integrity checks? What can they access? |
| Internationalization | See [4.19](#419-internationalization-and-localization) |

## 4.19 Internationalization and localization

Applies to: any code that produces text for people, or handles data in several languages.

| Checkpoint | What counts as a problem |
| --- | --- |
| Text externalization | User-facing text is hard-coded in the code and cannot be replaced |
| Plurals and grammar | Differences in plural rules, gender and word order between languages; assembling sentences by string concatenation |
| Dates, numbers and currency | Formats are hard-coded instead of following the locale; currency must not use floating point and must carry the currency code |
| Sorting and comparison | Collation depends on the language; case folding depends on the language |
| Text direction | Right-to-left languages; truncating and joining bidirectional text |
| Characters and length | The difference between bytes / code points / grapheme clusters / display width; can truncation cut a character apart? See [14](dimensions.md#14-boundaries-numbers-and-text) |
| Input normalization | Is input normalized? Comparison and deduplication of equivalent forms |
| Time zones and calendars | User time zone vs server time zone; non-Gregorian calendars |
| Fallback chain | Fallback order when a translation is missing; what it falls back to |

## 4.20 Concurrency primitives and general-purpose containers

Applies to: any synchronization primitives, collections, pools and rate limiters provided for others to use.

| Checkpoint | What counts as a problem |
| --- | --- |
| Contract completeness | Which methods may be called concurrently, whether they are reentrant, whether the zero value is usable, whether it may be copied: is all of this written down? |
| Copy safety | Is it still correct after being copied? Is there a way to prevent copying? |
| Fairness | Is fairness promised? If not, can starvation happen? |
| Generic constraints | Are the container's requirements on element types (comparable, hashable, non-null) written down? |
| Modification during iteration | What happens when items are added or removed during iteration? Is it a snapshot or a live view? |
| Capacity and growth | Growth policy, shrink policy, worst case |
| Zero value and empty instance | Operations on an empty container; method calls on an empty instance |
| Differences from built-in types | Compared with the language's built-in structure of the same kind, where does the behavior differ? Is it written down? |

## 4.21 Outbound requests and server-side request forgery

Applies to: any code that opens outbound connections or sends outbound requests based on external data (request parameters, stored records, configuration, redirects, webhooks, callback URLs). For code that only deals with connection pools, timeouts and certificates, see [4.1](#41-networking-and-connections); for inbound source addresses, see [4.14](#414-server-request-handling-and-middleware); for parsers fetching external entities, see [4.3](#43-decoding-untrusted-external-data).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Who decides the destination** | Check who actually controls the URL, host, IP and port; low-trust input that reaches the outbound path through configuration, storage or redirects is wrongly treated as trusted |
| **Protocol restrictions** | Allowing `file:`, `gopher:`, `dict:`, `unix:` or custom schemes that the target's job does not need; a protocol switch bypasses access restrictions that were already required |
| **Following redirects** | The first request is inside the allowlist, but after a redirect it reaches the internal network or switches protocol; no cap on the number of hops |
| **Time gap between resolving and connecting** | The allowlist checks the host name, but the real connection resolves it again → DNS rebinding |
| **Allowlist bypass** | Decimal / octal IPs, IPv4-mapped IPv6, `@` user info, homoglyphs, trailing dot, open redirect chains |
| **Internal network and metadata** | Can it connect to loopback, link-local, private networks, cloud metadata or container gateways in violation of the target's access policy? For components whose job is to proxy, check which restrictions the actual caller is responsible for |
| **Identity carried outbound** | Can internal credentials, cookies or origin tokens be carried to an address the user chose? |
| **Treating the response as trusted input** | The outbound response body is used again as a template, query, redirect target or command |
| Timeouts and size | Outbound requests can also be stalled by a slow upstream or a huge response; for timeouts see [4.1](#41-networking-and-connections), for amplification see [20](dimensions.md#20-resource-bounds-and-backpressure) |

For identity and replay checks on inbound webhooks, see [4.24](#424-webhooks-and-external-events).

## 4.22 Subprocesses, dynamic execution and decoder side effects

Applies to: any code that starts subprocesses, calls interpreters, uses eval, deserializes data into object graphs, loads plugins, or lets logging / template frameworks interpret lookup expressions. For parsing amplification and external entities, see [4.3](#43-decoding-untrusted-external-data).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Invocation method** | Building a shell command by concatenation; even with an argument array, an argument can still become an option or another file path |
| **Environment inheritance** | Can an untrusted party control the `PATH`, loader or debug variables, or working directory that the subprocess inherits, and so change the program and code that actually run? |
| **Descriptor leaks** | The parent process's key files and sockets are inherited by the subprocess |
| **Identity and privilege dropping** | External programs run with too many privileges; privileges are not dropped back on failure |
| **Object graph deserialization** | Untrusted bytes are unpacked as language objects → constructors / callbacks / gadgets get executed. This is a different kind of issue from "parser bombs" |
| **Decoder side effects** | Image / document / media libraries read local paths, reach the network or execute embedded configuration; for network access see [4.21](#421-outbound-requests-and-server-side-request-forgery) |
| **Logging and lookup expressions** | Untrusted messages trigger lookups, class loading or expression execution, turning an ordinary logging call into file access, network access or code execution |
| **Plugins and native libraries** | The load path or class name comes from external input |
| **Output reinterpreted** | The output of a subprocess or decoder is treated again as a command, as markup or as the next input |

## 4.23 Security audit logs

Applies to: applications, and their logging components, that need to trace sensitive access, permission changes, dangerous operations or security events. For general logging performance, see [4.13](#413-logs-metrics-and-tracing); for privacy constraints, see [30](dimensions.md#30-privacy-data-governance-and-compliance).

| Checkpoint | What counts as a problem |
| --- | --- |
| Event scope | Required events are missing, such as authentication failures, authorization denials, sensitive data access, permission and configuration changes, exports and deletions; general log levels or sampling accidentally turn off required records |
| Identity and correlation | Missing trusted principal, delegated principal, tenant, action, object, event time, result or correlation identifier; an identity self-reported in the request, or a forgeable source, is treated as a verified identity |
| Truthfulness of results | Success is recorded before the operation commits, or async execution has only an acceptance record and no final result; records cannot tell attempts, denials, partial completion and completion apart |
| Integrity and access | Business identities can change or delete records at will; transport, archives or queries lack access limits, so tampering and abnormal gaps cannot be detected |
| Write failures | Records are silently lost when the disk is full, the queue is full, the network is down or the logging service fails; the choice between blocking, rejecting business work or degrading has no contract and no alert; resending builds an unbounded backlog |
| Injection and data minimization | External fields forge new events or overwrite reserved fields, or full tokens, passwords and sensitive payloads are recorded; display and export interpret the log content again |
| Retention and disposal | Retention period, access responsibility and deletion rights are unclear; alerts cannot be linked to events that can be investigated; nobody notices when logging stops or audit configuration is turned off |

The content and strength of the records should follow the actual risk; you can check against the [OWASP Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html).

## 4.24 Webhooks and external events

Applies to: code that receives external callbacks or signed notifications, or changes business state driven by third-party events. For outbound callback URLs, see [4.21](#421-outbound-requests-and-server-side-request-forgery); for message delivery, see [4.10](#410-message-queues-and-async-jobs).

| Checkpoint | What counts as a problem |
| --- | --- |
| What is verified | Is the signature verified over the raw bytes or the canonical form the protocol requires? Middleware rewriting the content, duplicate fields, or parsing again after verification make the object that is acted on differ from the object that was signed |
| Trusted source | The verification key comes from trusted configuration and is bound to the right sender; a key carried in the message cannot replace it. A source IP restriction alone cannot prove that a message is genuine |
| Replay and concurrency | Are events outside the time window rejected? Is the event identity bound to the sender, tenant and purpose? Are the deduplication check and the business state change atomic? A timestamp alone cannot stop replay within the window |
| Business binding | Can a valid event be used for another tenant, order, resource or operation? Do state changes still check the current business preconditions? |
| Out of order and duplicates | Does an old event overwrite newer state? Do duplicate deliveries, manual replays and resends cause double charges, double grants or double notifications? |
| Acknowledgment timing | Is the event reliably saved or processed before success is returned? Can it recover as agreed after a crash during processing, a lost response or a downstream failure? |
| Reconciliation and gaps | For events the protocol allows to be lost or delayed, is there a check against the authoritative state or a compensation entry point? An external operation cannot be taken as completed just because the client was redirected with a success result |

## 4.25 Distributed coordination and leases

Applies to: code that relies on multiple nodes, distributed locks, leases, leader election or replication to make writes and jobs unique. For transaction and consistency promises, see [23](dimensions.md#23-transactions-atomicity-and-consistency).

| Checkpoint | What counts as a problem |
| --- | --- |
| Failure assumptions | Are network partitions, long pauses, clock skew, message delay and duplication covered by the promise? Multi-node safety cannot be derived from a single-process lock |
| Stale holders | After a lease expires or the leader changes, can the old holder still write when it comes back? When old writes must be excluded, does the storage side check an increasing fencing token or an equivalent condition? |
| Renewal and release | Does a failed renewal stop the operations that depend on the lease? Do release and retry check the holder's identity? Can an old holder delete a new lease? |
| Decision and commit | When a node gets a timeout but the commit result is unknown, does a retry execute the work twice? Do version numbers, terms or conditional writes cover the actual resource change? |
| Partition recovery | When several nodes each believe they may act, how are conflicts rejected? Do old snapshots, lagging replicas and replayed messages break results that were already confirmed? |

## 4.26 LLMs and tool calling

Applies to: code that uses large language models for retrieval, generation, agent execution or tool orchestration.

| Checkpoint | What counts as a problem |
| --- | --- |
| Instructions and external content | Are the sources of user tasks, retrieved documents, web pages, attachments, tool descriptions and return values kept apart? Can instructions inside external content change the task, tool permissions or where data goes? |
| Authorization at the execution layer | Does the tool entry point check the user, tenant, operation, parameters and resource scope on its own? Permissions are decided only by prompts or by another model, or model output can directly widen tool permissions |
| Read and output boundaries | Are retrieval, conversation memory, vector stores and caches isolated by principal? Does data sent to the model vendor or to tools go beyond what is authorized? Are URLs and code in the output executed directly? |
| Approval and actual action | Is an operation that needs approval bound to the final parameters, target and content? Do replacing parameters after approval, repeated retries or chaining tools widen the scope? |
| Call budgets and records | Do tool loops, recursive calls, model billing and external operations have budgets, cancellation and result records? A tool failure must not be recorded as success just because the model says the work is done |

For related defenses, see [OWASP LLM Prompt Injection Prevention](https://cheatsheetseries.owasp.org/cheatsheets/LLM_Prompt_Injection_Prevention_Cheat_Sheet.html); the actual authorization must still be enforced by the execution layer.

## 4.27 Interpreters, compilers and virtual machines

Applies to: code that implements a language, expressions, a query language, a template language, a regex engine or a bytecode virtual machine; check this especially when it runs untrusted source code. For the side that calls eval, see [4.22](#422-subprocesses-dynamic-execution-and-decoder-side-effects).

| Checkpoint | What counts as a problem |
| --- | --- |
| Semantic basis | Which specification, reference implementation and version is it measured against? Are deviations recorded? Is there differential testing against the reference implementation? |
| **Metering coverage** | Do budgets for steps, time, memory, depth and output size cover every syntax construct and built-in function? Can built-in functions (repeat, concatenation, sorting, regex, big-number arithmetic, string growth, collection expansion) bypass metering within a single call? |
| Compile-time cost | Are lexing, parsing, type checking, optimization and macro expansion themselves bounded against malicious source code (deep nesting, very long identifiers, exponential inference or expansion)? |
| **Host boundary** | Can scripts reach host objects, prototypes or reflection entry points? Can input lent to a script be modified by the script? Can a script still hold host references after it returns? State when a callback reenters the host |
| User callbacks | When the engine calls comparators, accessors or iterators provided by the script: what happens if a callback gives inconsistent results, throws, modifies the collection being processed, or reenters the engine? |
| Optimization equivalence | Do constant folding, inlining, type specialization and dead code elimination preserve semantics (NaN, negative zero, integer overflow, evaluation order, side effects, timing of exceptions)? |
| Run state isolation | When several runs or several tenants share one engine, are global objects, caches, prototypes, registers and pooled objects cleaned out? Can data from the previous script be read by the next one? |
| Value conversion | Width, precision, encoding and identity when numbers, strings and nulls are converted between host and script (across languages, see [4.32](#432-cross-language-boundaries-and-native-extensions)) |
| Errors and positions | Do syntax and runtime errors carry accurate positions? Do error objects leak host internals? Can source code passed in through a public entry point crash the engine internally? |
| Interruption and cancellation | Can a long-running script be interrupted from outside? Is the engine still usable after an interruption? |
| Compilation cache | Does the cache key include the source, version, options and host capabilities? Do different permission contexts share compiled results? |
| Determinism | Do the same source and input give the same result? Can random numbers, time and iteration order be controlled? |

## 4.28 Tunnels, proxies and the network data plane

Applies to: code that forwards other people's traffic, implements proxy protocols, VPNs or virtual network interfaces, transparent proxies, NAT or load balancing, or changes system routing and name resolution settings. For the connection layer, see [4.1](#41-networking-and-connections); for outbound destination policy, see [4.21](#421-outbound-requests-and-server-side-request-forgery).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Own traffic looping back** | Can this process's own outbound connections and name resolution be captured again by its own capture rules? What excludes them (marks, interface binding, route exceptions), and what happens when the exclusion fails? |
| **Leaks** | Do name resolution, IPv6, local multicast and direct-connection exceptions bypass the tunnel? Is there a brief cleartext direct connection while the tunnel is not ready or is reconnecting? |
| Failure direction | When the tunnel drops, configuration fails to load or the process crashes, is traffic blocked or does it fall back to a direct connection? Does that match the product promise (see [29](dimensions.md#29-fail-safe-behavior-and-dangerous-operations))? |
| **Leftover system state** | Do changed routes, name resolution settings, firewall rules and virtual interfaces remain after a crash or a forced kill? Can the next start recognize and clean up what the last run left behind? Can the cleanup delete someone else's rules by mistake? |
| **Open relay and reflection amplification** | Can unauthenticated parties use the service as an open proxy, open resolver or relay? Does a connectionless protocol send back more data than the request before the source address is verified? |
| Flow tables and mappings | Do connection tracking, NAT mappings and fake-address mappings have bounds, timeouts and reclamation? What happens when the table is full? Reused mappings mix up flows |
| Fragmentation, MTU and encapsulation overhead | Packets exceed the path MTU after encapsulation; bounds on fragment reassembly; black holes when path MTU discovery packets are dropped |
| Per-packet cost | Allocations, system calls and locking per packet; throughput and latency under bursts and small-packet floods |
| Authentication and replay | Is proxy protocol authentication bound to the session and direction? Can the handshake be replayed? With several users, are traffic and quotas isolated? |
| Protocol identification and probe resistance | Check only when the target promises resistance to identification: can unauthenticated probes recognize the service? Handshake fingerprints, packet length and timing features; does the response on authentication failure reveal what the service is? |
| Traffic splitting rules | When traffic is split or blocked by rules, see [4.29](#429-rule-and-policy-matching) |

## 4.29 Rule and policy matching

Applies to: code that allows, blocks, routes, rate-limits, authorizes or matches signatures based on a rule table, for example firewalls, access control lists, traffic splitting rules, WAFs and alert rules.

| Checkpoint | What counts as a problem |
| --- | --- |
| Match semantics | First match, longest match, by priority number, or evaluate all? Is it written down, and does the implementation agree? |
| **Default action** | When no rule matches, is it allow or deny? Does that fit the security goal? |
| Shadowing and conflicts | Rules fully shadowed by earlier rules that can never match; how conflicting rules are resolved |
| **Normalize before matching** | Are case, trailing dot, internationalized domain name encoding, IPv4-mapped IPv6, path encoding and Unicode normalization unified before matching? Is the representation used for matching the same as the one actually used (see [27](dimensions.md#27-security-and-trust-boundaries))? |
| **Boundaries** | Do domain suffixes match on label boundaries (`example.com` should not match `badexample.com`)? Can wildcards cross levels? CIDR containment, and whether port ranges are open or closed at each end |
| Regex and patterns | Missing anchors; anchor semantics change in multiline mode; dialect differences; backtracking cost (see [20](dimensions.md#20-resource-bounds-and-backpressure)) |
| Scale | Build time, memory and match cost with tens of thousands of rules; does an incremental update need a full rebuild? |
| Update atomicity | While the rule table is updated, does the matcher see the complete old table, the complete new table, or half a table? |
| Explainability | Can you find out which rule a request matched? |

## 4.30 OS interfaces, system calls and descriptors

Applies to: code that calls system calls or kernel interfaces directly (socket options, netlink, ioctl, device files) or platform native APIs, or that holds raw file descriptors or handles. For cross-language calls, see [4.32](#432-cross-language-boundaries-and-native-extensions); for platform differences, see [38](dimensions.md#38-portability-and-build-context).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Descriptor ownership** | After a raw descriptor or handle is wrapped in an object, who closes it? Does the wrapper close it as a side effect when it is collected? After a descriptor number is reused, is someone else's descriptor closed by mistake? |
| Interruption and retry | Are calls that were interrupted by a signal, are temporarily unavailable or only partly completed retried as the interface specifies? Treating them as final failures, or retrying forever |
| Error code semantics | An error code is valid only on failure and is overwritten by later calls; are expected states such as "already exists", "does not exist" and "permission denied" handled apart from real failures (see [13](dimensions.md#13-error-handling-and-failure-semantics))? |
| Struct layout and length | Layout, alignment, byte order and length fields of structs exchanged with the kernel or system libraries; is the length checked when reading variable-length messages returned by the kernel? |
| Blocking mode and inheritance | Blocking and non-blocking modes switched by accident; subprocesses inherit descriptors they should not (close-on-exec flag) |
| Thread-bound state | Network namespaces, thread-local system state, UI main-thread requirements; calls land on the wrong thread |
| Privileges | The message and degradation when a privileged call fails; are privileges dropped after the privileged operation (see [4.22](#422-subprocesses-dynamic-execution-and-decoder-side-effects))? |
| Signals | Only async-signal-safe operations inside signal handlers; signal masks and multithreading |
| Resource limits | Where do limits on descriptors, locked memory, process count and the like come from? What happens near a limit? |

## 4.31 Code generators and build-time tools

Applies to: tools that generate code and configuration from schemas, IDLs, templates or source code, and macros, plugins and annotation processors that run at build time. For the side that uses generated artifacts, see [39](dimensions.md#39-generated-artifacts-and-toolchain); for data contracts, see [4.16](#416-data-models-and-generated-contracts).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Input escaping** | Are names, comments and default values from the schema or template escaped before they go into generated code? Can they inject code, or end a comment or string early? |
| Name mapping | How are conflicts with target-language keywords, built-in names and other generated names handled? Is the mapping stable and reversible? |
| Output determinism | Does the same input produce byte-for-byte identical artifacts? Does it depend on unordered containers, timestamps, absolute paths or environment variables? |
| Input shape coverage | Does code generated for shapes such as optional, recursive, empty collections, deep nesting and union types compile and behave correctly? |
| Orphaned artifacts | Are old artifacts cleaned up after an input is deleted or renamed? Does the cleanup delete only files the tool generated? |
| Generator version | Do artifacts record the generator version? Can the diff after a generator upgrade be reviewed? |
| Build-time execution | What can the generator, macro or plugin access at build time (network, files, environment variables)? Is its input trusted? |
| Verification | Are the generation rules or configuration contracts tested? Do artifact snapshots cover representative inputs? |

## 4.32 Cross-language boundaries and native extensions

Applies to: repositories that mix several languages, and code that works across languages through foreign function interfaces, native extensions, embedded runtimes, WebAssembly, subprocesses, inter-process communication or shared schemas.

| Checkpoint | What counts as a problem |
| --- | --- |
| Boundary inventory | List every cross-language boundary and its direction: who calls whom, how data is passed, who owns memory and resources |
| **Type mapping** | Are integer width and sign, floating point, booleans, nulls, string encoding (UTF-8, UTF-16, NUL-terminated), time and time zones, and unknown enum values consistent on both sides? |
| **Memory and lifetime** | Who frees pointers or buffers that cross the boundary, and with which allocator? After the garbage-collected side hands memory to native code, can that memory be moved or collected? Does native code quietly keep callbacks or pointers? |
| **Errors and exceptions across the boundary** | Can exceptions or panics cross the boundary (many combinations are undefined behavior or terminate the process outright)? How are error codes and exceptions converted into each other, and is context lost? |
| Concurrency and threads | Acquiring and releasing the global interpreter lock (GIL); which thread runs callbacks; do the native code's thread-safety assumptions match the host's? |
| Build and distribution | Compiler, ABI and platform matrix for native extensions; source and verification of prebuilt binaries (see [36](dimensions.md#36-supply-chain-and-artifact-integrity)); can the library search path at run time be hijacked? |
| Single source of definitions | Are the data definitions on both sides generated from the same schema? Do two hand-written definitions agree (see [4.16](#416-data-models-and-generated-contracts), [5](dimensions.md#5-coupling-and-blast-radius))? |
| Trust direction | When the memory-unsafe side handles untrusted input, can its errors break the guarantees of the memory-safe side? |
| End-to-end tests | Are there tests that run through both sides of the boundary, or does each side test only against its own test doubles (see [41](dimensions.md#41-test-quality) "Test double drift")? |

## 4.33 Build scripts, CI and infrastructure as code

Applies to: build scripts and packaging configuration, CI and release workflows, container image definitions, deployment manifests, and infrastructure as code. For the supply chain as a whole, see [36](dimensions.md#36-supply-chain-and-artifact-integrity); for the runtime environment, see [34](dimensions.md#34-runtime-environment-and-deployment-contract).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Workflow injection** | Fields from events (title, branch name, commit message, comments) are spliced directly into scripts or commands |
| **Privileged triggers** | Code from external contributors is checked out and run in a context that has secrets or write permission |
| Token permissions | Are the default permissions of workflow tokens and cloud credentials narrowed to the minimum? Have short-lived credentials been adopted? |
| Pinning third-party steps | Are referenced external actions, images and scripts pinned to immutable digests, rather than movable tags or branches? |
| Secret exposure | Do secrets end up in logs, artifacts, image layers, caches, or contexts visible to external contributors? |
| Images | Running with the highest privileges; base image not pinned; credentials written into image layers at build time; downloaded files not verified |
| Deployment manifests | Privileged containers, host directory mounts, host networking, no resource limits, no health checks, the default service account |
| Infrastructure permissions and exposure | Public storage buckets, security groups open to the whole internet, overly broad roles; secrets in state files |
| Drift and reproducibility | Does the declaration match the real environment? Manual changes get overwritten by the next apply, or overwrite the declaration the other way round |
| Script robustness | Does the build script stop on errors (for Shell details see [Appendix A](lang-shell.md))? Order dependencies in parallel builds; deletion scope of cleanup steps |

## 4.34 Publishable libraries, SDKs and packages

Applies to: units that others install, reference or link as a package, library, SDK, plugin or shared library.

| Checkpoint | What counts as a problem |
| --- | --- |
| Public surface | Are the symbols, entry points and subpaths actually exported exactly the ones meant to be public? Can internal implementation be referenced directly from outside? |
| **Breaking changes** | Are changes to public interfaces, binary interfaces, default behavior and error types consistent with what the version number promises? Is there a diff check? |
| Standalone build | Can it build and test outside the repository workspace and without local overrides (see [38](dimensions.md#38-portability-and-build-context))? |
| Declared vs actual | Do the declared minimum runtime, platforms and dependency ranges match what the code actually needs? A range that is too wide pulls in versions nobody tested; one that is too narrow conflicts with users |
| Published contents | Does the package carry test data, secrets, local paths or build caches? Are all the needed files included? |
| Install and load-time behavior | Does it run code, reach the network or change global state at install, import or load time? |
| Good manners toward the host | Does it modify process-wide globals (default client, logging configuration, signal handling, environment variables, global random seed)? Does it exit the process directly (see [13](dimensions.md#13-error-handling-and-failure-semantics), [37](dimensions.md#37-interoperability-and-coexistence))? |
| Types and metadata | Are type declarations, license, repository URL and deprecation markers complete and accurate? |
| Multiple copies side by side | When two versions or two module formats of the same library are loaded at once, do global state and type checks split? |

## 4.35 Client apps, extensions and auto-update

Applies to: apps that run on user devices, such as desktop apps, mobile apps, browser extensions and resident clients. For the user interface itself, see [4.18](#418-user-interfaces-and-accessibility).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Auto-update** | Are update packages signed and verified before installation? Can the update channel be downgraded or replaced? Can a failed update be rolled back? |
| **External invocation** | Can custom protocols, deep links, exported components and inter-process interfaces be invoked by other apps or web pages? Are the arguments passed in validated? |
| **Local control plane** | Are local ports, local sockets and named pipes open only to the intended principals? Can web pages reach them through localhost or DNS rebinding (see [4.14](#414-server-request-handling-and-middleware))? |
| Local secrets | Are tokens and keys kept in the system credential store, or in plain-text files? Can backup and sync carry them off the device? |
| Embedded web content | Local interfaces that embedded browser components expose to web pages; where loaded content comes from; navigation restrictions |
| Permissions | Are the requested system permissions minimal? Behavior when a permission is denied |
| Lifecycle | State recovery after the system suspends the app, kills it or reclaims it for low memory; limits on running in the background |
| Offline and poor networks | Behavior while offline; conflict handling after reconnecting |
| Crash reports and telemetry | Do reports include personal information (see [30](dimensions.md#30-privacy-data-governance-and-compliance))? Can users turn them off? |

## 4.36 Embedded, firmware and real-time constraints

Applies to: code that runs on microcontrollers, in firmware or in drivers, or that has real-time deadlines.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Interrupt context** | Are operations in interrupt handlers reentrant and bounded? How is data shared with the main loop protected? Does the compiler know that these variables can change asynchronously? |
| Stack and static memory | Has the worst-case stack depth been computed? When dynamic allocation is banned or limited, is that rule followed? |
| Timing | Worst-case execution time vs deadlines; priority inversion; busy waiting and watchdogs |
| **Power loss and writes** | Power loss while writing flash; write endurance and wear leveling; is the configuration area stored twice with checksums? |
| Firmware update | Update package signature; A/B partitions and rollback; can the device still boot after an interrupted update? |
| Peripherals and registers | Order of register access and memory barriers; peripheral timeouts; workarounds for hardware errata |
| Debug interfaces | Are debug ports and debug output turned off in shipped products? Is read protection enabled? |
| Environmental limits | Behavior at the limits of memory, storage and battery; environmental anomalies such as low voltage and temperature |

## 4.37 Data processing, batch jobs and machine learning

Applies to: batch jobs, data pipelines, stream processing, analysis notebooks, and model training and inference code.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Idempotent reruns** | When a job is rerun after a failure or a partial success, does it write or count twice? Is output written to a temporary location first and then published? |
| Data contract | When the input schema changes, fields go missing or types drift, does it raise an error, skip, or silently produce wrong results? |
| Late, duplicate and out-of-order data | Time windows, watermarks and deduplication in stream processing |
| Scale and skew | Memory and run time as data volume grows and keys become skewed; loading the full data set at once |
| Training data leakage | Leakage between training and test sets; features that use information not available at prediction time |
| Reproducibility | Are random seeds, data versions, dependency versions and hardware differences recorded? |
| Model and data files | Does loading a model or data file run code (see [4.22](#422-subprocesses-dynamic-execution-and-decoder-side-effects))? Is the source trusted? |
| Hidden notebook state | State left by running cells out of order; data or credentials embedded in outputs |
| Personal data | Minimization, redaction and deletion propagation for personal information in the pipeline (see [30](dimensions.md#30-privacy-data-governance-and-compliance)) |
| Inference boundaries | Behavior when input falls outside the training distribution; confidence scores treated as facts |

## 4.38 Smart contracts and on-chain interaction

Applies to: contracts deployed on a blockchain, and off-chain services that sign or broadcast transactions or read on-chain data.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Reentrancy** | External calls happen before state updates; cross-function and cross-contract reentrancy |
| Arithmetic and precision | Overflow checks, rounding direction of division, conversion between token decimals |
| **Access control** | Are admin, initialization and upgrade functions protected? Can someone else call initialization first? |
| Oracles and prices | Using prices that can be manipulated within a single transaction; stale data |
| Transaction ordering | Front-running and sandwich attacks; slippage and deadline parameters |
| External call results | Return values are not checked; a transfer to an address that rejects it reverts everything, causing denial of service |
| Upgrades and storage layout | Storage slot collisions after proxy upgrades; differences between initializer functions and constructors |
| Gas and loops | Loops that grow with the number of users; permanently unable to execute once the block gas limit is exceeded |
| Off-chain signatures and keys | Are signatures bound to the chain ID, contract address, nonce and expiry time? Storage and spending limits of hot wallet keys (see [4.4](#44-cryptography-and-credentials)) |
| Reorgs and confirmation | Off-chain services change business state based on blocks that are not yet final; is that rolled back after a chain reorganization? |
