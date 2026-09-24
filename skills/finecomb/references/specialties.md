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
| **Connection state after cancellation** | When a request is cancelled or times out after it was sent but before the response was fully read, is the connection simply discarded? If a connection returned to the pool still holds an unread response, the next request (possibly from another user) reads the result of the previous request |
| Upgrading to encryption mid-stream | For protocols like STARTTLS that start in cleartext and then switch to encryption: are cleartext commands buffered before the upgrade discarded after it? When the upgrade fails or the peer does not support it, does it refuse to continue rather than fall back to cleartext? More generally, can any data received before the security layer (TLS handshake, authentication, encrypted channel) is established be treated as data inside the security layer? |

## 4.2 Protocols and frame parsing

Applies to: any code that parses bytes off the wire itself and keeps protocol state.

| Checkpoint | What counts as a problem |
| --- | --- |
| Message boundaries | Several messages in one read / one message split across reads; what happens when one read returns half a frame, or two and a half frames? |
| **Coexisting with another parser** | Your code and the upstream / downstream disagree on length, chunking or line endings → request smuggling — see [4.14](#414-server-request-handling-and-middleware) |
| **Unknown must-understand items** | Extensions, header parameters and options that the sender marks as "critical" or "must understand" (such as critical certificate extensions, `crit` in JWS, or critical control items in a protocol) are just ignored and accepted as usual when the receiver does not recognize or implement them; the constraint the sender required stops working, and the result disagrees with what a strict implementation decides |
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
| **Handler chosen by the request** | When the parser or handler is chosen by attributes the request controls (content type, extension, parameters), can switching to another representation skip the path that does sanitization or limiting, while the later point of use is still reached? |
| External entities and references | Will the parser fetch external references (local files, remote addresses)? For outbound destinations see [4.21](#421-outbound-requests-and-server-side-request-forgery) |
| Partial decoding | Side effects already produced when decoding fails |
| Repeated structures | When a segment, block, header or field that the format allows only once appears again: is it rejected, replaced as a whole, or does it overwrite only part of the state, so that buffers and counts, or pointers and lengths, that should be paired come from different instances? |
| Object merging and prototype pollution | Can special properties of an external object, through merging, assignment by path or copying, modify a shared prototype, class or global object (such as the JavaScript prototype chain, or the Python attribute chain through `__class__` and `__init__.__globals__`), and then affect permissions, configuration or later requests? |
| Multiple formats and duplicate fields | When the same bytes can be read as different objects by different parsers, do validation and use apply the same format, field precedence and handling of invalid input? |
| Decoders you depend on | Known vulnerabilities of third-party decoding libraries; are their dangerous features disabled? |
| **Decoding becomes execution** | Deserialization runs code; the decoder reads local files or reaches the network — see [4.22](#422-subprocesses-dynamic-execution-and-decoder-side-effects) |

## 4.4 Cryptography and credentials

Applies to: any code that uses or implements cryptography: encryption, message authentication, hashing, signatures, key exchange, certificates and tokens, secret sharing, commitments, multi-party computation and threshold signatures, Merkle proofs and other kinds of proofs, remote attestation, and any code that touches keys, tokens or passwords. The checkpoints are organized by "facet", not by algorithm, and apply to every algorithm and construction; for zero-knowledge circuits, see also [4.44](#444-zero-knowledge-proofs-and-circuits).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Algorithms and constructions** | Deprecated or weak algorithms are used (MD5 or SHA-1 still used for security purposes: in 2012 Flame used an MD5 collision to forge a code-signing certificate, and in 2017 SHAttered produced a SHA-1 collision); the mode of operation is wrong (ECB, or CBC or CTR without authentication); encryption without authentication, or, when "encryption + MAC" is assembled by hand, it is not encrypt-then-MAC, or the MAC does not cover the IV and the header; using `H(key ‖ message)` for authentication is open to length extension, so use HMAC or a dedicated keyed hash; a non-cryptographic hash is used for security integrity checks, and a non-secret checksum cannot replace message authentication or a signature; common AEADs do not commit to the key, so the same ciphertext can decrypt successfully under two different keys, and multi-recipient, abuse reporting and deduplication cases need a committing scheme |
| **Randomness and unpredictability** | Do keys, tokens, salts and signature nonces come from a cryptographic random source? Is a failure of the random source ignored? For ECDSA, Schnorr and DSA, a reused, biased or length-leaking signature nonce can each reveal the private key (in PuTTY CVE-2024-31497 the top 9 bits of the P-521 nonce were always 0, and about 60 signatures were enough to recover the private key; Minerva and TPM-FAIL leaked the nonce length through timing), so nonces should be generated deterministically per RFC 6979 or taken from a reliable random source; does the key generation process have structural flaws (ROCA)? |
| **Inputs to deterministic nonces** | The inputs used to derive the signature nonce are not the same values the signing equation actually uses (the message integer after truncation or reduction, the public key, the context), or cannot be mapped back to them one to one; type conversion, sign, truncation or non-canonical encoding lets two different values being signed get the same nonce; the signing function accepts messages of any type instead of rejecting them. Using RFC 6979 does not by itself make it safe; check the inputs to the derivation |
| **Uniqueness and single use** | Do IVs and nonces meet the uniqueness or unpredictability requirements of the algorithm used? Can concurrency, restarts, multiple instances, restoring backups or counter wraparound cause reuse? Reusing a nonce with GCM or ChaCha20-Poly1305 not only leaks plaintext but also lets an attacker forge messages (a 2016 scan found more than a hundred HTTPS servers repeating GCM nonces); the collision bound for random nonces, the usage limit for one key, and when to rotate keys; do values that "can be used only once" (challenges, nonces, nullifiers, key images) really have only one valid value? |
| **Authorized operations that refer to objects by position** | A signed, approved or queued operation names its object by index, position, sequence number or "current value" instead of by identity; when it runs on another chain, in another account or after the state has changed, can it land on a different object? For operations that are deliberately allowed to run across chains, across accounts or later, is the object's identity checked at execution time? |
| **Binding and context** | Do signatures, MACs, additional authenticated data, proofs and key derivation cover the fields, operation, tenant, recipient, direction, protocol purpose and version that are actually used? Do canonicalization ambiguity, cross-purpose reuse or unsigned fields change the meaning of what gets executed? Does the handshake bind the full negotiation transcript (against downgrade, splicing and unknown key-share)? Is a shared secret passed through a KDF again with both parties' identities and the protocol context mixed in? After a certificate, token or attestation report passes verification, is the result bound to this session and purpose? |
| **Domain separation and purpose separation** | The same key is used for several purposes or for both directions of communication; key derivation lacks a purpose label; when the same hash or signature is used in several contexts, does the input carry a purpose prefix, so that a result from one context cannot be used in another? Do Merkle tree leaf nodes and internal nodes use different prefixes (otherwise a second preimage can be forged)? Can the rule of duplicating the last node when the count is odd make different sets produce the same root (Bitcoin CVE-2012-2459)? |
| **Canonical form and malleability** | Before structured data is signed or hashed, is it first turned into a single canonical form, and do the signer and the verifier use the same rules? Is the encoding of signatures, keys, curve points and certificates strict (strict DER, low s values, values smaller than the group order, compressed versus uncompressed points)? Can several valid encodings of the same value be treated as different transactions, records or identifiers? Can a signature or proof itself be transformed into another equally valid value, so that it cannot serve as a unique identifier or an anti-replay key? |
| **Completeness of verification** | When verifying signatures, certificate chains, tokens or proofs, is every item checked: the signature, the chain, the validity period, basic constraints, key usage, name constraints, the host name, the issuer, the audience, who decides the algorithm, and extensions and header parameters marked as critical or must-understand (see [4.2](#42-protocols-and-frame-parsing) ("Unknown must-understand items"))? When several checks write to the same result, a later success must not turn an earlier failure back into success. For WebAuthn, the relying party ID, the origin and the freshness of the challenge; can an error path in a verification function be treated as success (Apple "goto fail" CVE-2014-1266, GnuTLS CVE-2014-0092), and is the return value used correctly? Are public keys and curve points supplied from outside checked to be on the curve and in the correct prime-order subgroup, and are an all-zero shared secret and the point at infinity rejected? On curves whose cofactor is not 1 (such as ed25519), can one secret be used to build several equivalent points that all pass verification (Monero 2017)? When a revocation check fails, does it allow or reject? |
| **Parameters supplied by the other party** | Are the algorithms, keys, moduli, curves and group parameters given by the protocol peer, by multi-party computation participants or by token headers validated, or accompanied by a correctness proof that is verified (without a proof for the Paillier modulus, a malicious participant can steal other parties' key shares within a dozen or so signatures: BitForge CVE-2023-33241)? Can `alg`, `kid`, `jku` and `x5u` in a token be chosen only from a trusted set? |
| **Misuse of signing interfaces** | Can the signing function accept a public key passed in separately by the caller (with Ed25519, signing the same message with one private key paired with different public keys leaks the private key; several libraries were affected in 2022, such as CVE-2022-50237)? Do BLS aggregate signatures require proof of possession of the public keys (against rogue public keys)? Nonce commitments and concurrent sessions in multi-party signing; is the signature verified over the message or over a precomputed hash, and is the hash algorithm bound to the signature? |
| **Oracles and observable differences** | When decryption, signature verification, password comparison or unwrapping fails, can the error type, timing or response length tell the causes of failure apart (padding oracles, the Bleichenbacher attack and the 2017 ROBOT, Lucky13 CVE-2013-0169)? Error messages leak "which step failed"; when attacker-controlled data is compressed together with a secret and then encrypted, the ciphertext length leaks the secret (the CRIME and BREACH class); does streaming decryption hand plaintext to the caller before authentication has passed? |
| **Constant time and side channels** | Keys, authentication codes or tokens are compared with a short-circuiting comparison; secret-dependent branches, memory accesses, divisions (KyberSlash 2024: secret-dependent division in several ML-KEM implementations leaked the key) and early returns; do compiler optimizations break constant time? Cache leaks from table-lookup implementations on shared CPUs; power and electromagnetic analysis on embedded devices; voltage or clock glitches make checks get skipped, and a faulty RSA-CRT signature leaks the private key, so is the result verified after signing? |
| **Key derivation and passwords** | A password is used directly as a key (no key derivation); derivation parameters are too weak; does derivation use a dedicated KDF with a salt and a purpose label? Are the password hashing algorithm and cost parameters strong enough, and can they be upgraded? A separate salt for each password; when a function has an input length limit (for example bcrypt looks only at the first 72 bytes), is the excess silently dropped? When several fields are concatenated before derivation, a long enough earlier field pushes the password behind it out (Okta 2024) |
| Key lifecycle and custody | Do key generation, storage, use, rotation, revocation and destruction form a closed loop? Are the acceptance period for old keys and the behavior on rotation failure clearly defined? Layering in envelope encryption, and re-encrypting old data after rotation; do the interfaces of HSMs, smart cards and TPMs (such as PKCS#11) allow keys to be exported or wrapped into an exportable form? Key material is exported without authorization, or goes in plain text into unprotected storage, transport or object representations |
| Credential exposure | Do source code, repository history (read as authorized), configuration, samples, logs, errors, image layers, cache keys or crash dumps contain live secrets? When one is found, record only the redacted location and the impact, and do not spread the original value; deleting a copy does not mean the credential has been revoked |
| Secrets in memory | Are secrets zeroed after use, and can the compiler remove the zeroing? Can they be paged out to disk, or written into crash dumps, core dumps or debug snapshots? |
| Protocol state and replay | Are handshake messages rejected when they arrive out of order, are repeated or are skipped? Record sequence numbers and rekeying on wraparound; can session resumption and 0-RTT data be replayed? After a long-term key leaks, are past sessions still safe (forward secrecy)? |
| **Custom schemes and proof structure** | Do home-grown or composed cryptographic protocols and proof systems have a security analysis and a dedicated audit, and does the implementation match the paper or specification item by item? When an interactive proof is made non-interactive (Fiat–Shamir), does the hash for each challenge include every value the prover has already sent before that challenge is derived (commitments, claimed evaluation results, sub-challenges the prover chooses itself), plus all public inputs and parameters (a missing item lets proofs be forged: the Frozen Heart issue in several implementations in 2022)? In secret sharing, are share indexes unique and non-zero, and are shares verifiable? The binding and hiding properties of commitments; can threshold and multi-party computation schemes identify a misbehaving party? Lower and upper bounds in range proofs; do proofs of reserves include all liabilities and negative balances? Does proof of work check the target and the full header? Verification of verifiable random functions and verifiable delay functions |
| Cryptographic agility and post-quantum | Can algorithms, key lengths and parameters be replaced without changing the protocol? Does data that must stay secret for a long time face the risk of "harvest now, decrypt later"? Implicit rejection when post-quantum decapsulation fails; how the two keys are combined when a post-quantum algorithm is used in hybrid with a classical one |
| Remote attestation | Attestation reports from a TPM or trusted execution environment: signature chain, allowlist of measurements, freshness (nonce), and whether the platform security version is still supported; are secrets released after attestation passes bound to this attestation? |
| **Secrets shipped to clients** | Do frontend bundles, mobile apps, desktop installers or public configuration endpoints carry server-side keys or high-privilege tokens? Anything shipped to a user's device is not a secret |
| Secrets passed on the command line | Are secrets passed to subprocesses as command-line arguments, where other users on the same machine can see them in the process list, or where they end up in shell history and audit logs? |

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
| Delegated authorization protocols | When OAuth, OpenID Connect, SAML or challenge-response authentication is used, check each item in [4.46](#446-federated-identity-and-single-sign-on) |
| **One-time codes** | Verification codes sent by SMS or email, or generated as one-time passwords: are attempts counted per account and per code, not only per session or source address (switching session or address lets the attacker start over)? Are the validity period and time window short? Is a code invalidated after it is used or replaced by a newer one? Is it bound to this specific login or operation? |
| Push approvals | Can push-based multi-factor prompts be triggered again and again to bombard the user? Does the approval screen show enough about where the request comes from to recognize it, and does it require typing the number shown on screen (number matching)? Is there an alert after repeated denials? |
| Identity granted by email domain | Is "automatically joining an organization or gaining permissions because the email belongs to a certain domain" based on a verified email? Do the side that validates and the side that delivers mail parse email addresses the same way (quotes, comments, encoded words, multiple @)? |
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
| Cache keys and isolation | Do keys include the tenant, principal, permissions or representation dimensions that decide the response? Do negative results, shared object pools or error responses leak into other requests? Raw credentials must not be used as keys; do keys built from several variable-length fields have length prefixes or separators? |
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
| Query injection | Are data values parameterized? Do structures that cannot be parameterized, such as table names, column names and sort expressions, use an allowlist? Do ORM raw queries, NoSQL operators and search expressions still accept arbitrary input? When escaping must be done by hand, see [4.17](#417-templates-and-text-output) ("Escaping under multibyte encodings") |
| **Drivers that emulate parameterization** | In some modes (simple query protocol, emulated prepared statements, client-side interpolation) the driver or ORM inlines parameters into the statement text; safety then depends on the literals the driver generates: do negative numbers, special floating-point values, arrays and custom types all produce literals that carry their own boundaries? Is this mode on by default, and can configuration turn it on? |
| **Connection strings and driver parameters** | When a database connection string, DSN or driver parameters are decided by external or low-privilege input, can driver features (initialization scripts, loading local files, automatic deserialization, custom socket factories) run scripts, read local files or connect to any host? Is the connection target restricted by an allowlist (for outbound requests see [4.21](#421-outbound-requests-and-server-side-request-forgery))? |
| Data-layer permissions | Does the database account have more privileges than its job needs? With row-level policies or a connection-level tenant context, can a reused connection keep the identity of the previous request? |
| Migrations | Are they reversible? Do they lock tables? Strategy for adding columns / adding indexes / changing types on large tables; ordering relative to code releases |
| Nulls and defaults | Mapping between nulls in storage and nulls in the language; which layer holds default values |
| Time and time zones | Does the stored time type carry a time zone? Are the same time zones used for writing and reading? |
| Read/write splitting | Can reads from a replica return stale data? Can read-your-writes be guaranteed? |
| Dialects and implicit behavior | Three-valued logic of nulls, implicit type conversion, collation and trailing-space comparison, truncation in non-strict mode, and default isolation levels differ between databases; for details see [the SQL section of Appendix A](lang-sql.md) |
| **External objects used as query conditions** | Objects or key-value pairs from the request are expanded directly into filter conditions (ORM keyword arguments, `where` objects, search expressions), so attackers can filter on password hashes, tokens or fields of related tables and work out data they cannot see one character at a time |
| Directories and other query languages | LDAP filters and DNs have different escaping rules; is each escaped with its own rules? Are XPath, XQuery and search engine query syntax parameterized or built from an allowlist? |

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
| Path length and characters | Length limits, illegal characters, reserved names, case, normalization forms; on Windows, alternate data streams (`name::$DATA`), 8.3 short file names, and trailing dots and spaces can defeat extension checks and blocklists |
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
| Long-running flows and orchestration | Can workflows, sagas and orchestration instances that run for days continue across a code upgrade? Are compensation steps idempotent? How are stuck instances found and handled? |

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
| **Security response headers** | Are HSTS, content security policy, X-Content-Type-Options, Referrer-Policy and framing protection (frame-ancestors) set according to what each page is for? For cookie attributes see [4.5](#45-authentication-sessions-and-tokens) |
| HTTP caching semantics | Are Cache-Control, Vary, ETag and conditional requests correct? Are responses that carry identity marked as cacheable by shared caches? Can path confusion make a sensitive page get cached as a static resource (cache deception)? Can a request make a dynamic response that should not be cached carry cacheable headers, and so poison a shared cache? |
| GraphQL | Limits on query depth and complexity; amplification from aliases and batched queries; is introspection open to outsiders? Field-level authorization (see [28](dimensions.md#28-authorization-and-access-control)) |
| gRPC and other RPC | Message size limits; are deadlines passed downstream? Are the reflection service and health checks exposed to outsiders? Does the mapping of error statuses leak internal information? |
| **API inventory** | Is there an inventory of externally reachable APIs, old versions, deprecated APIs and debug APIs? Are "shadow APIs" that are not in the inventory still serving, and do they bypass checks added in newer versions? |
| Request methods | Are authorization and CSRF checks attached only to some methods (only GET and POST are checked, while HEAD or unknown methods still reach the handler)? Are headers or parameters that override the method, such as `X-HTTP-Method-Override`, accepted? |
| Cross-site leaks and request origin | Can cross-site pages infer user state from load results, timing, frame counts or error events? Are `Sec-Fetch-*` request headers used to reject requests that should not be loaded cross-site, and COOP and CORP used to limit cross-origin access to windows and resources? Is JSONP turned off? |

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
| Configuration discovery | Does the program look for configuration, plugins or hooks in parent directories or the current directory (repository directories, workspace settings, `.env`)? In shared directories or in projects handed over by someone else, can these files make the program run commands? |
| **Implicit expansion by argument-parsing libraries** | Argument-parsing libraries by default replace "@file" with the file's contents (response files), or expand wildcards or environment variables; when a server, remote interface or privileged process reuses a command-line parser to handle untrusted arguments, are these expansions turned off? |

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
| **Escaping under multibyte encodings** | When the escaper splits characters by character encoding, how does it handle invalid or truncated multibyte sequences? Do the escaper and the downstream parser agree on where characters begin and end? Can the backslash the escaper adds, or a quote that should have been escaped, be absorbed into the preceding multibyte character so that the quote takes effect again? When agreement cannot be guaranteed, is input with invalid encoding rejected? |
| Context-aware escaping | The same data needs different escaping in attributes, scripts, styles and URLs |
| **Boundaries at the insertion point** | Escaping based only on the value's own content, without wrapping the value in its own boundary (quotes, brackets, a length prefix); the first and last characters of the value combine with the template characters just before and after the insertion point into a new token (two `-` become a line comment, `/` and `*` become a block comment, a value that starts with `-` becomes an option); types that "need no escaping", such as numbers and booleans, are concatenated directly |
| User content | User-provided content is parsed as a template or as markup |
| Template source | Does the template itself come from an untrusted source? |
| Output size | Is the expanded output capped? |
| Line breaks and encoding | Injected line breaks break the structure (log injection, header injection, and line-based internal protocols such as memcache, Redis and SMTP); can terminal control characters fake the diagnostic display? Output encoding declaration |
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
| **Client-side security** | Do DOM writes and dynamic execution accept unprocessed data? Do the sanitizer and the browser parse the same markup the same way (mutation XSS)? Are the content security policy, framing protection, and the origin, sender and structure checks on cross-window messages effective? For cookies see [4.5](#45-authentication-sessions-and-tokens); for CORS and CSRF see [4.14](#414-server-request-handling-and-middleware) |
| Scope of directives set by responses | When implementing a browser or web engine: are directives that are set by response headers or content and can relax security defaults (referrer policy, content security policy, cross-origin policies, permissions policy) applied only to the requests the specification says they apply to? Can a response controlled by an attacker extend them to cross-origin subresource requests? |
| Browser persistent state | Do local storage, offline caches and Service Workers keep data from the previous account? Can it still be read after logout, tenant switching or revoked authorization? |
| **Open redirect** | Can external input make a redirect go beyond the allowed range, disguise itself as a trusted destination or carry credentials out? A redirect back after login must also meet the authentication binding in [4.5](#45-authentication-sessions-and-tokens) |
| Third-party scripts | Do included external scripts have integrity checks? What can they access? What happens if the domain serving the script changes hands or the CDN is taken over? |
| DOM clobbering | Can user-controlled markup (`id` and `name` attributes) override global variables or `document` properties that scripts read, and so change where scripts load from or their configuration? Does the sanitizer handle this case? |
| **Injected UI covered by the host page** | For UI that this component injects into someone else's page (autofill, consent dialogs, prompts), can the host page make it transparent, shrink it or put it under other elements, so that the user's clicks or consent land where the attacker wants? |
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
| **Credentials across redirect chains** | Authentication headers, cookies and proxy authentication stripped on a cross-host redirect: are they restored when a later hop comes back to the same host or to the original host? Is the stripping decision based on the original request or on the previous hop? Do tests cover chains of three or more hops? |
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
| **Windows command-line reassembly** | On Windows, an argument array is joined back into a single command line, and the called program splits it again by itself; when the target is a `.bat` or `.cmd` file, it always goes through `cmd.exe`, and commands can still be injected even when arguments are escaped as an array |
| **Character set conversion rewriting arguments** | When wide-character arguments and paths are converted to the local code page, the system applies approximate mappings (fullwidth quotes become `"`, soft hyphens become `-`, `¥` becomes `\`); validation happens before the conversion, while execution sees the result after it |

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
| **Secrets in prompts** | Do system prompts or tool descriptions contain keys, internal addresses or rules that should not be public? Can users coax them out? Security must not depend on keeping prompts secret |
| Output reliability | Are the model's answers, citations and calculation results used directly as facts? Is there source checking, a confidence level or human confirmation? |
| Model and data sources | Where do model files, fine-tuning data and retrieval corpora come from, and are they intact? Can externally writable corpora be poisoned and affect later answers? |
| Regression evaluation | After changes to prompts, model versions or retrieval configuration, is there a fixed evaluation set that confirms behavior has not regressed? |
| **Rendered output that calls out** | When model output is rendered as Markdown or HTML, can images, reference-style links or link previews automatically request external addresses, carrying data from the context out in the URL? Can the domains allowed by the content security policy themselves forward requests? |
| **Rewriting its own constraints** | Can the agent write to files or settings that decide its own permissions (auto-approve switches, tool allowlists, hooks, project configuration)? Must such writes be confirmed by a person (for the general criterion see [28](dimensions.md#28-authorization-and-access-control) ("Permission ceiling of derived identities"))? |
| Persistent memory | Can external content be written into long-term memory, user profiles or a shared knowledge base, and keep taking effect in later sessions or for other users? Do memory entries have a source, a principal and a validity period? |
| Tool definitions and their origin | Can a tool's description and parameter descriptions change after approval? Can a tool with the same name shadow a tool from another server? Is re-approval required when a definition changes? |
| Credentials held by agents | When an agent or tool server calls downstream services, does it use credentials narrowed to the user and the purpose, forward the token it received as is, or act with its own high-privilege identity? Can the model read the credentials? |
| Messages between agents | Is output from other agents, subagents and workflow nodes treated as external content? Do messages carry a sender identity that can be verified? |

For related defenses, see [OWASP LLM Prompt Injection Prevention](https://cheatsheetseries.owasp.org/cheatsheets/LLM_Prompt_Injection_Prevention_Cheat_Sheet.html); the actual authorization must still be enforced by the execution layer.

## 4.27 Interpreters, compilers and virtual machines

Applies to: code that implements a language, expressions, a query language, a template language, a regex engine or a bytecode virtual machine; check this especially when it runs untrusted source code. For the side that calls eval, see [4.22](#422-subprocesses-dynamic-execution-and-decoder-side-effects).

| Checkpoint | What counts as a problem |
| --- | --- |
| Semantic basis | Which specification, reference implementation and version is it measured against? Are deviations recorded? Is there differential testing against the reference implementation? |
| **Metering coverage** | Do budgets for steps, time, memory, depth and output size cover every syntax construct and built-in function? Can built-in functions (repeat, concatenation, sorting, regex, big-number arithmetic, string growth, collection expansion) bypass metering within a single call? |
| Compile-time cost | Are lexing, parsing, type checking, optimization and macro expansion themselves bounded against malicious source code (deep nesting, very long identifiers, exponential inference or expansion)? |
| **Host boundary** | Can scripts reach host objects, prototypes or reflection entry points? Can input lent to a script be modified by the script? Can a script still hold host references after it returns? State when a callback reenters the host; can exception objects, error stack handling hooks or callback arguments bring host objects into the script? |
| **Process-wide state** | Can scripts in trusted mode or in a sandbox change the host process's global state (environment variables, working directory, locale, signal handling, loaded modules, global configuration)? Do these changes stay after the script ends and then get used by the host with its own privileges (starting subprocesses, loading libraries, resolving paths)? |
| User callbacks | When the engine calls comparators, accessors or iterators provided by the script: what happens if a callback gives inconsistent results, throws, modifies the collection being processed, or reenters the engine? |
| Optimization equivalence | Do constant folding, inlining, type specialization and dead code elimination preserve semantics (NaN, negative zero, integer overflow, evaluation order, side effects, timing of exceptions)? |
| Run state isolation | When several runs or several tenants share one engine, are global objects, caches, prototypes, registers and pooled objects cleaned out? Can data from the previous script be read by the next one? |
| Value conversion | Width, precision, encoding and identity when numbers, strings and nulls are converted between host and script (across languages, see [4.32](#432-cross-language-boundaries-and-native-extensions)) |
| Errors and positions | Do syntax and runtime errors carry accurate positions? Do error objects leak host internals? Can source code passed in through a public entry point crash the engine internally? |
| Interruption and cancellation | Can a long-running script be interrupted from outside? Is the engine still usable after an interruption? |
| Compilation cache | Does the cache key include the source, version, options and host capabilities? Do different permission contexts share compiled results? |
| Determinism | Do the same source and input give the same result? Can random numbers, time and iteration order be controlled? |
| Timer precision and side channels | Can scripts get high-precision timers or shared memory and use them for side channels such as speculative execution? When several tenants share a process, is process isolation needed? |
| **Assumptions behind speculative optimization** | After code is specialized by type, shape or length, or checks are removed, if a callback (getter, `valueOf`, proxy, destructor) or another execution flow changes the object, does the optimized code keep running on the old assumptions? When an assumption no longer holds, is there deoptimization or a recheck? |
| **Range analysis and check elimination** | For bounds checks and overflow checks removed by type inference or range analysis, is the inference itself right (integer wraparound, negative zero, NaN, lengths that can change)? One wrong inference becomes an out-of-bounds read or write |
| **Internal sentinel values leaking out** | Can special values the engine uses internally to mean "hole", "uninitialized" or "no exception" reach the script through exceptions, iteration, serialization or error paths? |
| Garbage collection and raw references | Around points where allocation, garbage collection or compaction can happen, are raw pointers and unregistered handles still used? Are write barriers missing in generational or incremental collection? |

## 4.28 Tunnels, proxies and the network data plane

Applies to: code that forwards other people's traffic, implements proxy protocols, VPNs or virtual network interfaces, transparent proxies, NAT or load balancing, or changes system routing and name resolution settings. For the connection layer, see [4.1](#41-networking-and-connections); for outbound destination policy, see [4.21](#421-outbound-requests-and-server-side-request-forgery).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Own traffic looping back** | Can this process's own outbound connections and name resolution be captured again by its own capture rules? What excludes them (marks, interface binding, route exceptions), and what happens when the exclusion fails? |
| Exclusion criteria borrowed by others | Can a managed process or workload obtain for itself the criterion that lets this process's own traffic bypass capture or policy (running UID, cgroup, firewall mark, source port, bound interface), and so bypass the whole isolation? |
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
| Real-time media and relays | Can relays (TURN) for real-time communication such as WebRTC be used as open proxies to reach the internal network? Do ICE candidates leak internal addresses? Validity period and scope of relay credentials |

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
| **Handles and capabilities passed across boundaries** | For handles, capabilities or shared memory references passed to or received from a less trusted process (sandbox, renderer process, guest, plugin) through inter-process communication or messages: is it confirmed that they point to the expected object and carry only the least privilege? Can logic errors, reuse, races or messages crafted by the other side let a wrong or over-privileged handle cross the isolation and be used? |
| Interruption and retry | Are calls that were interrupted by a signal, are temporarily unavailable or only partly completed retried as the interface specifies? Treating them as final failures, or retrying forever |
| Error code semantics | An error code is valid only on failure and is overwritten by later calls; are expected states such as "already exists", "does not exist" and "permission denied" handled apart from real failures (see [13](dimensions.md#13-error-handling-and-failure-semantics) ("Failures overwritten by later steps"))? |
| Struct layout and length | Layout, alignment, byte order and length fields of structs exchanged with the kernel or system libraries; is the length checked when reading variable-length messages returned by the kernel? |
| Blocking mode and inheritance | Blocking and non-blocking modes switched by accident; subprocesses inherit descriptors they should not (close-on-exec flag) |
| Thread-bound state | Network namespaces, thread-local system state, UI main-thread requirements; calls land on the wrong thread |
| Privileges | The message and degradation when a privileged call fails; are privileges dropped after the privileged operation (see [4.22](#422-subprocesses-dynamic-execution-and-decoder-side-effects))? |
| Signals | Only async-signal-safe operations inside signal handlers; signal masks and multithreading |
| Resource limits | Where do limits on descriptors, locked memory, process count and the like come from? What happens near a limit? |
| **Reading untrusted memory twice** | The same data is read twice from user space, shared memory or another process's buffer, with the first read used for validation and the second for use (double fetch), and the other side changes it in between |

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
| Self-hosted runners | Do workflows of public repositories run on self-hosted runners? Is each runner thrown away after a single job, or can external commits leave a backdoor on it and read secrets from later jobs? |

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
| Vulnerability intake and support scope | Are the vulnerability reporting channel, supported versions and fix timelines written down? After a fix, is a security advisory published that names the affected versions? |

## 4.35 Client apps, extensions and auto-update

Applies to: apps that run on user devices, such as desktop apps, mobile apps, browser extensions and resident clients. For the user interface itself, see [4.18](#418-user-interfaces-and-accessibility).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Auto-update** | Are update packages signed and verified before installation? Can the update channel be downgraded or replaced? Can a failed update be rolled back? |
| **External invocation** | Can custom protocols, deep links, exported components and inter-process interfaces be invoked by other apps or web pages? Are the arguments passed in validated, and can they be taken as the program's own command-line switches? |
| **Addresses handed to the system to open** | Before an address from the peer, a web page or a model is handed to the system's default program to open (`open`, `start`, `xdg-open`, `shell.openExternal`), is it limited to expected schemes such as `https:`, and does it avoid going through a command interpreter? |
| **Download origin marks** | Do files that are downloaded, extracted, saved under a new name, or taken out of email or chat carry the system's origin mark (Windows Mark-of-the-Web, the macOS quarantine attribute) over to the new file? If extracted files, temporary copies or shortcuts miss the mark, the system no longer warns or checks signatures |
| **Integrity coverage** | Do code signing and integrity checks cover every file that is loaded and executed at run time (snapshots, sidecar data, plugins, scripts, configuration), or only the main program? Can files that are not covered be modified locally? |
| Extension message sources | Are messages from content scripts treated as untrusted input? Are `externally_connectable` and external message handling limited to specific origins? Can web pages use the extension's high-privilege APIs to send requests or read data? |
| Extension permissions and exposed resources | Are host permissions narrowed to the sites that are needed? Are `web_accessible_resources` kept to a minimum (web pages can probe and embed these resources)? Does it load remote code? |
| Inter-process calls in desktop frameworks | When the main process handles messages from renderer processes, does it check the sender page? Does the preload script expose the whole inter-process communication object or Node APIs to web pages? Are navigation and new windows restricted? Are unneeded fuses turned off at packaging time (see [4.47](#447-local-privileged-components-and-local-privilege-escalation))? |
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
| **Functional safety** | When the safety of people or equipment is involved: is there a hazard analysis? Can it enter a safe state on errors? Redundancy, watchdogs and self-tests; are the requirements of the applicable functional safety standards met? |
| **Anti-rollback** | Can old images, bootloaders or drivers that are signed but have known vulnerabilities be accepted again? Does the security version number only ever increase, and is it stored where attackers cannot change it? Is the revocation list updated together with the fix? |
| Memory protection and lock bits | Do memory protection unit or page table regions overlap, or does their order let a lower-priority region cover a protected one? Are lock bits set after configuration is finished, and do they stay in effect after reset and after waking from sleep? Are the reset values of security-related registers safe? |

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
| Fairness and bias | Have differences in the results of models or rules across groups of people been evaluated? Is the training data representative? |

## 4.38 Smart contracts and on-chain interaction

Applies to: contracts deployed on any blockchain, written in any contract language, and chain modules that call contracts or are called back by contracts (such as Cosmos SDK modules). For DeFi economics, see [4.41](#441-defi-economics-and-oracles); for cross-chain interaction, see [4.42](#442-cross-chain-bridges-and-messaging); for wallets, signing and off-chain components, see [4.43](#443-wallets-signing-and-off-chain-components); for zero-knowledge proofs, see [4.44](#444-zero-knowledge-proofs-and-circuits); for nodes and consensus, see [4.45](#445-blockchain-nodes-consensus-and-protocol-implementations); for account abstraction and smart contract wallets, see [4.50](#450-account-abstraction-and-smart-contract-wallets). For the pitfalls of each contract language, see the [Solidity and Vyper](lang-solidity.md), [Solana](lang-solana.md) and [Move](lang-move.md) tables in Appendix A. When only a contract address is given, see [Targets that are not source code](targets.md#on-chain-contract-addresses).

| Checkpoint | What counts as a problem |
| --- | --- |
| Specification and invariants | Are the protocol's core invariants (conservation of totals, solvency, shares matching assets) written down? Are they covered by invariant tests, fuzz tests or formal verification? |
| **Reentrancy** | External calls happen before state updates; cross-function and cross-contract reentrancy |
| Arithmetic and precision | Overflow checks, rounding direction of division, conversion between token decimals |
| **Access control** | Are admin, initialization and upgrade functions protected? Can someone else call initialization first? |
| Oracles and prices | Using prices that can be manipulated within a single transaction; stale data |
| Transaction ordering | Front-running and sandwich attacks; slippage and deadline parameters |
| External call results | Return values are not checked; a transfer to an address that rejects it reverts everything, causing denial of service |
| **Arbitrary calls** | Does the contract, on the caller's behalf, call a target address with call data that the caller chooses (routers, aggregators, batch executors, cross-chain adapters)? Can the token approvals users gave this contract be used through such a call to run `transferFrom` and take other people's assets? Is there an allowlist for call targets and functions? |
| Upgrades and storage layout | Storage slot collisions after proxy upgrades; differences between initializer functions and constructors |
| **Upgrade procedure** | Does the upgrade run all the initialization the new version needs (such as `reinitializer`)? Variables that are missed stay at zero. Does the new implementation still keep the upgrade entry point? Is the storage layout before and after the upgrade compared item by item? Is the upgrade transaction rehearsed first in a forked environment? |
| Upgrade patterns | UUPS: does `_authorizeUpgrade` check permissions, and does the new implementation still carry the upgrade function? Beacon proxies: one upgrade affects every proxy; who owns the beacon? Diamond (EIP-2535): permissions on `diamondCut`, function selector collisions between facets, and the initialization parameters of `diamondCut` run code at an arbitrary address through `delegatecall` |
| Proxies and constructors | Storage written by the implementation contract's constructor is not visible to the proxy; `immutable` values are written into the implementation contract's code, so all proxies share the same value; does the implementation contract call `_disableInitializers` in its constructor? |
| Storage layout evolution | Are the order and types of variables, the inheritance order and the reserved storage gaps (`__gap`) the same before and after an upgrade? Has it switched to namespaced storage (ERC-7201)? |
| **Deployment scripts and parameters** | Is each dependency address (oracles, tokens, routers) correct for each chain? Are deployment and initialization done in the same transaction? After deployment, are ownership and every role handed over to a multisig or timelock? Do testnet and mainnet parameters differ? |
| **Deployed version versus audited version** | Does the on-chain bytecode match the audited commit? Have code added or changed after the audit, and newly launched modules and routes, been audited? Contracts outside the audit scope that are still called |
| Gas and loops | Loops that grow with the number of users; permanently unable to execute once the block gas limit is exceeded |
| **Signatures and replay** | Are off-chain signatures (permits, meta-transactions, orders) bound to the chain ID, contract address, nonce and deadline? Do they use domain-bound structured signing (such as EIP-712)? Signature malleability; is the signature rejected when verification returns the zero address or an empty result? For keys and the signing flow, see [4.43](#443-wallets-signing-and-off-chain-components) |
| Reorgs and confirmation | Off-chain services change business state based on blocks that are not yet final; is that rolled back after a chain reorganization? On chains with few validators, blocks already confirmed may later be rolled back by agreement |
| Non-standard token behavior | Are tokens that charge a fee on transfer, have balances that change on their own, return no value, have callbacks, have a blacklist or can be paused, or do not use 18 decimals all handled by the amount actually received? |
| More non-standard tokens | Reverts when an allowance is changed from non-zero to non-zero (it must be set to zero first); reverts on zero-amount transfers or zero-amount approvals; reverts when an amount exceeds `uint96`; the same balance has several contract entry points, so protections keyed on the token address can be bypassed through another entry point; upgradable tokens and tokens that can be flash-minted; native coins that also have an ERC-20 form; passing the maximum value transfers only the whole balance |
| **Privilege holders** | Are the admin, owner, upgrader and pauser a single externally owned account, a multisig or a timelock? The multisig threshold and how the signers are spread; can privileges be renounced or transferred, and does a transfer need two-step confirmation? |
| **Handing over privileges** | Do deployer, developer or test addresses still hold some role, proxy admin rights or upgrade rights? Is each one checked against the role grant and revoke events, instead of looking only at `owner()`? |
| Emergency mechanisms | Do pausing, limits and circuit breakers exist? Who can trigger them and who can lift them? Can users withdraw their assets while paused? If an upgrade goes wrong, is there a way to stop losses without waiting for the full timelock, and are that mechanism's own permissions limited? |
| Cross-chain address assumptions | The same address on another chain does not necessarily belong to the same person: when a multisig or smart contract wallet has not been deployed on the target chain yet, can someone else deploy to that address first? Before sending to such an address, is it confirmed that it is deployed and has the same owner? |
| On-chain randomness services | When a verifiable random function (such as Chainlink VRF) is used: are requests and results matched by request ID? Is the number of confirmation blocks enough to resist reorgs? Is re-requesting or canceling allowed? Is user input still accepted after the request is sent? Can the callback revert (the service does not retry)? |
| On-chain data is public | All storage, call data and events on chain can be read by anyone, `private` variables included; plain text, passwords and answers in commit-reveal schemes must not go on chain directly |
| Merkle airdrops and claims | What are claims deduplicated by (index, address or leaf)? Is the library version used for multi-proofs affected by known flaws? Who generates the Merkle root, and can it be replaced? |
| Forked code | Is code forked from a well-known protocol compared change by change against upstream? When a constant is changed, are all the places that depend on it changed too? Are vulnerabilities that upstream fixed later brought over? |
| Realistic test environment | Have tests run on a mainnet fork with real tokens, oracles and dependent protocol state? Can mock objects hide the real behavior of non-standard tokens and external contracts? Have tests run on the target chain's testnet? |
| Assumptions of verification | Are the assumptions of formal verification and invariant tests (loop unrolling bound, simplified external calls, excluded input ranges) written down? When the preconditions contradict each other, a rule passes for any implementation (a vacuous rule); has this been checked? |
| External dependency contracts | Are the addresses of the oracles, routers, tokens and other protocols it depends on hard-coded or changeable? What happens to this protocol when they are upgraded or compromised? |
| Account model assumptions | Using `tx.origin` or code length to decide that "the other side is an externally owned account"; since EIP-7702, externally owned accounts can also carry code, so such checks are no longer reliable |
| Compiler and target chain | Is the compiler version pinned? Is it affected by known compiler bugs? Does the target chain support the opcodes the compiler emits? Differences when the same code is deployed to several chains |
| On-chain observability | Do key state changes emit events? Are event fields enough for monitoring and indexers to rebuild the state? Are there alerts for unusually large outflows? |

## 4.39 Payments, accounting and billing

Applies to: code that handles payments, refunds, balances, points, bills, invoices or billing. For the numbers and rounding of amounts, see [14](dimensions.md#14-boundaries-numbers-and-text); for bypassing flows, see [9](dimensions.md#9-business-logic-and-flow-integrity).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Ledger invariants** | Is double-entry bookkeeping or an equivalent conservation constraint used? Do the debits and credits of every transaction balance? Can balances be recomputed from the journal? |
| Immutable journal | Are posted records append-only, never modified or deleted? Are corrections made through reversal entries? |
| **Idempotent charges** | Does each call to a payment provider carry an idempotency key? Can a retry after a timeout or a lost response charge twice? When the callback and an active status query disagree, which one wins? |
| Reconciliation | Is there regular reconciliation against payment providers and bank statements? How are differences found, recorded and handled? |
| Amounts and currencies | Amounts use integers in the smallest unit or fixed-point decimals; the currency is stored with the amount; the source, time and rounding rules of exchange rates |
| Price snapshots | Are the price, discounts and tax rate at order time saved as a snapshot? Can later price or rule changes alter orders that are already completed? |
| Refunds and chargebacks | Can partial refunds, repeated refunds and chargebacks add up to more than the original payment? Do refunds go back through the original payment method? How refunds interact with shipping and points |
| **Concurrent balance updates** | Can concurrent deductions overdraw? Are the frozen, unfrozen and expired states consistent (see [23](dimensions.md#23-transactions-atomicity-and-consistency))? |
| Bills and invoices | Are numbers sequential and unique? Can they be changed after they are issued? Is tax rounded per line or on the total? |
| Subscriptions and metering | Boundaries and time zones for renewals, upgrades and downgrades, trial expiry and usage-based billing; how lost or duplicate metering data is handled |

## 4.40 Notifications and outbound messages

Applies to: code that sends email, SMS, voice calls, push notifications or in-app messages, or calls back to external addresses. For templates and escaping, see [4.17](#417-templates-and-text-output); for outbound destinations, see [4.21](#421-outbound-requests-and-server-side-request-forgery).

| Checkpoint | What counts as a problem |
| --- | --- |
| **SMS and toll fraud** | Can attackers trigger SMS or voice verification codes in bulk to premium-rate numbers or to large numbers of phone numbers, so costs run out of control? Are there limits per number, country, account and source? |
| Used as a spam relay | Can email or messages with user-controlled content be sent to arbitrary unverified addresses, so others use it to send spam or phishing? |
| Header and content injection | Can line breaks in the recipient, subject or sender name inject email headers? Are links and content in messages escaped? |
| Timing of sending | The notification has already gone out after the transaction rolls back; retries cause duplicate notifications; the state in the message differs from the final state |
| Unsubscribe and consent | Do marketing messages have an unsubscribe option, and is it honored? For consent records, see [30](dimensions.md#30-privacy-data-governance-and-compliance) |
| Bounces and invalid addresses | Are bounces, complaints and invalid push tokens handled, to avoid sending again and again and hurting sender reputation? |
| Sender identity | Are SPF, DKIM and DMARC configured for the sending domain? Where are the credentials for push and SMS channels stored? |
| Content leaks | Do notifications carry sensitive information they should not? What is exposed in lock screen previews, forwarding and CC? |

## 4.41 DeFi economics and oracles

Applies to: on-chain financial protocols such as lending, exchanges (automated market makers and order books), yield aggregators, stablecoins, derivatives and staking. For general contract checks, see [4.38](#438-smart-contracts-and-on-chain-interaction).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Price manipulation** | Do prices come from sources that a single transaction can manipulate (the spot reserves of an automated market maker, the latest trade in a thin market)? Can a flash loan borrow, manipulate, profit and repay within one transaction? |
| **Oracle data** | Are oracle prices checked for update time and validity? On layer 2 networks, is sequencer uptime checked? What happens when several price sources disagree or fail? |
| Time-weighted prices | Is the window of the time-weighted average price long enough? With low liquidity, can it be manipulated steadily across several blocks? |
| **Shares and rounding** | Does the rounding direction of share calculations on deposit and withdrawal favor the protocol? Can the first depositor raise the share price by direct donation, so that later depositors' shares round down to zero (vault inflation attack)? |
| **New markets and empty pools** | New lending markets, vaults or pools go live while their totals are zero or very small: are creation, opening deposits and borrowing, and setting collateral factors done in the same transaction? Are initial shares that can never be withdrawn deposited before launch? After a share price or exchange rate is pushed up by direct donation, can it be used as the price of collateral? |
| **Asset listing and risk parameters** | When low-liquidity assets (especially the protocol's own token) are used as collateral, are the collateral factor, supply cap and borrow cap set according to market depth? Can the price be pushed up with a small amount of money? Can repeated deposit-and-borrow loops inflate the collateral shares? |
| **Price feed selection and conversion** | Is the feed's trading pair exactly the asset being valued (wrapped assets, liquid staking tokens and pegged coins cannot simply use the price of the underlying asset)? Feeds differ in precision and update interval; is each one handled separately? Does the application set its own reasonable price bounds, instead of relying on minimum and maximum values in the oracle contract that are no longer used? |
| Pull-based oracles | Oracles where users bring their own price updates (such as Pyth): users can pick a price in their favor within the allowed time range and submit it; is price freshness limited? Is the confidence interval checked? Is the exponent conversion correct? |
| Front-running price updates | When trades or derivatives fill immediately at the current oracle price, can someone see a price update that is about to land on chain and place orders before it? Is two-step ordering used (submit a request first, then execute at a later price)? |
| Liquidation | Liquidation trigger conditions, rewards and discounts; can liquidation happen in time during sharp price moves or network congestion? How is bad debt handled? |
| Liquidation edge cases | Liquidation continues while repayment and adding collateral are paused; the liquidation reward on small positions does not cover gas, so nobody liquidates them and bad debt piles up; a position can be liquidated right after borrowing; can liquidators liquidate only the part that benefits them? |
| Interest rates and accrual | Precision and update timing of interest index accrual; after a long time without updates, does settling everything at once overflow or become inaccurate? |
| Governance | Can voting power be borrowed temporarily with a flash loan? Is there a timelock between proposal and execution? Permissions of the emergency proposal channel |
| **Quorum and the cost of buying control** | Is the number of votes needed to pass a proposal calculated from the total supply or from actual participation? When participation is low, can a single address get over the line on its own? Is the cost of buying or borrowing these votes on the market far lower than the treasury and permissions the proposal can use? Is there a veto, a guardian or a cap on large transfers as a backstop? |
| **Timelocks** | Do all privileged operations go through the timelock? Is there a role that can bypass it? Can the delay be set to 0? Can users exit before a change takes effect? Who can execute queued operations, and can an operation that is split into several transactions be executed first by someone else in a different order? |
| Checking what a proposal executes | Is the code reviewed at voting time the same code that runs at execution time? Can the contract a proposal points to be upgraded or have its code replaced after the vote? Is the proposal's call data decoded and checked independently? |
| Front-running and transaction ordering | Slippage protection, deadlines, commit-reveal; can transactions in the public mempool be sandwiched? Trust assumptions of private transaction channels |
| Source of slippage parameters | Is the minimum output given by the caller off-chain? Computing the quote inside the contract from the same pool that can be manipulated, writing it as 0, or setting the deadline to `block.timestamp` gives no protection at all |
| Pool hooks | Pools with hooks (such as Uniswap v4): are hook callbacks allowed only from the pool manager? When anyone can create pools with your hook, is state kept separate per pool ID, and are allowed pools restricted? Do the permission bits in the hook address match the actual implementation? Can a failed optional external call block users from exiting? State crosstalk when several pools share one hook |
| Permissionless integration | For markets, pools, tokens or vaults that anyone can register, is the data they return (reward token lists, prices, balances) treated as trusted? Can registered external contracts reenter during callbacks? |
| Staking and validators | How are the withdrawal queue and slashing of a staking pool spread across shares? When depositing to the beacon chain on behalf of users, can a node operator front-run with a deposit that uses its own withdrawal credentials, so the stake ends up under its own name? |
| Incentives and economic attacks | Can rewards be farmed through wash volume or self-trading, or extracted by depositing and withdrawing within one block? Have parameter changes gone through economic simulation? |
| Composability risk | Knock-on effects when external protocols it depends on (tokens, lending pools, routers) fail, pause or are compromised; read-only reentrancy reading prices from intermediate state |

## 4.42 Cross-chain bridges and messaging

Applies to: contracts that move assets or pass messages between chains (including messages between layer 1 and layer 2), and off-chain relayers and validators.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Message verification** | How does the target chain prove that a message really happened on the source chain: light clients, Merkle proofs, validator signatures? After initialization or an upgrade, can the trust root and default values (such as an all-zero root) make any message count as verified? |
| **Validators and thresholds** | Can the number of validators or multisig signers, the threshold and the key distribution withstand a few leaked keys? Who can change the validator set? |
| **Messaging-layer security configuration** | When a third-party messaging layer (such as LayerZero, Wormhole, CCIP, Hyperlane) is used, how many independent verifiers does each path require, and what is the threshold? Is this configured explicitly, instead of relying on defaults that the messaging layer may change? Are the send and receive configurations on both ends consistent? Who can change the configuration? |
| **Receive entry point checks** | Does the function that receives messages check that the caller is the messaging layer's endpoint or router, and also check that the source chain and source sender are on the allowlist? If only one of these is checked, someone can send a forged message from another chain or another contract |
| Replay and uniqueness | Can the same message run more than once on the target chain? Is it bound to the source chain, target chain, contract address and sequence number? When a source chain message is signed before it is final, the same sequence number may map to different content after a reorg, so deduplicating by sequence number is not enough |
| Execution failure on the target chain | When execution on the target chain fails because it runs out of gas or reverts, is the message stuck, retryable or lost? Who can retry, and can the retry use a different gas amount? In an ordered channel, does one failure block all later messages? How are assets returned after a failed message expires? |
| Layer 1 to layer 2 address aliasing | For messages that a layer 1 contract sends to layer 2, the sender seen on layer 2 is an alias address with a fixed offset added; does authentication compare against the alias? Do Starknet layer 1 handler functions check `from_address`? |
| Sequencer downtime and forced inclusion | When the layer 2 sequencer is down or censoring, users can only submit through layer 1 with forced inclusion, which takes a long time to take effect; what happens to deadlines, liquidations, auctions and oracle staleness checks during that time? Is there a grace period after recovery? |
| Layer 2 withdrawal delay | Withdrawals from optimistic rollups must wait for the challenge period; does the protocol treat "withdrawal started" as "funds arrived"? |
| Forged accounts and parameters | Are the system accounts, contract addresses and parameters that the verification flow depends on checked for their real identity? A forged system account can make a signature check look as if it passed |
| Asset matching | Do the amounts of lock and mint, burn and release always match? Is there monitoring of conservation of totals? |
| Finality | Does the target chain let things through while the source chain is not yet final or is going through a reorg? |
| Limits and pausing | Outflow caps per transaction and per unit of time; can it be paused when something goes wrong? Is the pause permission itself safe? |

## 4.43 Wallets, signing and off-chain components

Applies to: code and processes that manage private keys, organize signing, provide frontends, or run off-chain services (relayers, indexers, bots, oracle nodes).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Key custody** | Where are private keys or seed phrases stored (hardware security modules, multi-party computation, plain-text files, environment variables)? Who can access them? The backup and recovery process; what tool or firmware generated the private keys and seed phrases, and when that tool or firmware has a known randomness flaw, can the affected addresses be identified? |
| **Blind signing** | Is what the signer sees on the signing device really what gets signed? Can the frontend or an intermediate service swap the displayed transaction for a different one (for example, replacing a transfer with a delegate call that upgrades the contract)? |
| Multisig process | Before signing, are the transaction's call data, target and operation type checked in an independent environment? Is there transaction simulation and a target allowlist? Does each signer compute the transaction hash on an independent device and compare it with the hash the hardware wallet shows? |
| **Pre-signed and long-lived signatures** | How long do transactions, approvals or delegations that are signed but not yet executed stay valid (Solana durable nonce transactions, permits with distant deadlines, delegation authorizations with chain ID 0, multisig signatures collected off-chain)? Can they be revoked? Are signers asked to sign, in advance, admin operations that "will only be executed later"? |
| Frontend and hosting | If frontend code, DNS, the CDN or storage buckets are tampered with, will users sign malicious approvals? Integrity and deployment permissions of frontend assets; if a third-party library the frontend uses (such as a wallet connection library) is poisoned, can it pop up signing requests directly? Are these dependencies pinned to fixed versions and integrity-checked? |
| Address display and poisoning | Does the interface show only the first and last few characters of an address? Can users copy look-alike addresses and zero-amount transfers that others forged into the history? Is there an address book and a warning for look-alike addresses? |
| Approvals and permits | Is the token allowance requested from users minimal? The impact of unlimited approvals and off-chain permit signatures after phishing; can users view and revoke them? |
| Nodes and data sources | Are the RPC nodes and indexing services it relies on trustworthy? Consequences when returned data is tampered with or lagging; is there cross-checking across several sources? |
| Off-chain bots and relayers | Hot wallet limits and how they are topped up; handling of stuck transactions, fee caps and nonce conflicts; behavior when front-run |
| Keepers | What happens to the protocol when the off-chain programs that liquidation, settlement and price feed updates depend on stop or fall behind? For keeper entry points that anyone can trigger, can someone pick the timing, order and parameters that benefit them? |

## 4.44 Zero-knowledge proofs and circuits

Applies to: implementations of zero-knowledge circuits, proof systems and verifiers, including private transactions, zero-knowledge rollups, zkVMs, on-chain verifier contracts and off-chain proving services. For the specific pitfalls of circuit languages, see the [Zero-knowledge circuits](lang-zk.md) table in Appendix A; for general cryptography checks, see [4.4](#44-cryptography-and-credentials).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Under-constrained circuits** | Is every witness value in the circuit bound by constraints to public inputs, constants or other constrained values? A cell that is only assigned and not constrained lets a malicious prover choose any value while verification still passes (in 2026, Zcash Orchard allowed unlimited forging of ZEC because the base point of scalar multiplication was only assigned, not constrained) |
| **Malicious-prover tests** | Do tests use only correct witnesses produced by an honest prover? Are there negative tests of the form "change any one witness value or swap a bound value, and verification must fail"? Has the constraint set been checked for completeness with under-constraint detection tools or formal methods? |
| **Deriving unique identifiers** | Are the unique identifiers used against double spending (nullifiers, key images and so on) derived from the secret and the note with full constraints in the circuit? Can one note produce several unique identifiers that are all valid? |
| **Value conservation** | Does the circuit or the consensus layer constrain input and output amounts to balance? Do amounts have range constraints (so finite field wraparound cannot turn a negative number into a huge amount)? Is there supply accounting independent of the proofs (such as tracking inflows and outflows per pool) as a backstop? |
| **Public input binding** | Which public inputs does the proof bind? Does the verifier compute or check these inputs itself, instead of accepting whatever values the prover gives? Can the same proof be reused with a different set of public inputs? |
| Canonical form of inputs | Are public inputs, field elements and curve points reduced to a unique representation before verification? Can a value x and x plus the field modulus both pass verification, giving the same identifier two forms? |
| **Challenges and transcripts** | Before each challenge is derived, has every value the prover has already sent been absorbed into the transcript: commitments, claimed evaluation results, sub-challenges in the proof chosen by the prover (such as the branch challenges of an OR proof), and all public inputs and system parameters? List by round "what the prover has sent, and what is in the transcript at this point"; if one item is missing, the prover can see the challenge first and then choose that value (missing items allow forged proofs; this is the Frozen Heart issue found in several proof system implementations in 2022) |
| **Trusted setup and parameters** | For schemes that need a trusted setup, who took part in the parameter generation ceremony, and how was the "toxic waste" destroyed? Can the parameters and verification keys be verified, and do they match the circuit version (CVE-2019-7167, disclosed by Zcash in 2019, allowed forged proofs because of extra elements in the parameters)? |
| Proof malleability | Can a proof be transformed into another equally valid proof? A proof must not be used as a unique identifier or a replay protection key |
| Verifier implementation | On-chain verifier contracts and off-chain verifier implementations: input validation for pairings and point operations, return values of precompile calls, and whether the verification key is hard-coded and matches the circuit |
| Recursion and aggregation | In recursive or aggregated verification, are the inner proof's verification key, public inputs and circuit identity constrained by the outer layer? |
| zkVM and execution traces | Do the constraints on the execution trace cover the full semantics of every instruction? Memory read/write consistency checks; constraints on system calls and precompiles |
| Compilers and toolchains | Versions and known flaws of circuit compilers, constraint generators and proof libraries; do the optimized constraints match the semantics of the source? |
| Prover and verifier consistency | Do the prover side, the verifier side and each client use the same circuit version, parameters and hash implementation? How are old proofs handled when the circuit is upgraded? |

## 4.45 Blockchain nodes, consensus and protocol implementations

Applies to: blockchain nodes, consensus engines, transaction validation, peer-to-peer networking, and protocol implementations such as wallet cores and light clients. For the contract layer, see [4.38](#438-smart-contracts-and-on-chain-interaction); for zero-knowledge parts, see [4.44](#444-zero-knowledge-proofs-and-circuits).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Consensus determinism** | Every node must compute the same result from the same input: floating-point math, iteration over unordered containers, dependency library versions, local time and concurrency order can all cause divergence; differences between client implementations lead to chain splits |
| **Supply and double-spend checks** | Duplicate inputs within a transaction or a block; overflow when adding amounts; calculation of fees and block rewards; are these checks run on every validation path (mempool, blocks, replay after a reorg)? (In 2010, Bitcoin's CVE-2010-5139 amount overflow created about 184 billion BTC out of thin air; in 2018, CVE-2018-17144 came from skipping the duplicate input check within blocks for performance) |
| Consistent mempool and block paths | Is the validation on entering the mempool the same as the validation when packing into a block and when receiving a block? Are there checks that exist on only one path? |
| Peer-to-peer messages | Are the size, count and computation cost of peer messages capped? Can malicious nodes exhaust memory, CPU, connections or disk? Can they cut a node off from the honest network (eclipse attack)? |
| Upgrades and activation | Activation height or conditions of forks, and the boundary where old and new rules switch; behavior of nodes that have not upgraded; the switch and permissions for emergency soft forks |
| Canonical signatures and encodings | Can non-canonical encodings of signatures, transactions and blocks be accepted, making transaction IDs malleable? Replay protection (chain ID) |
| Time and difficulty | Can block timestamp rules and difficulty adjustment be manipulated (time warp)? Logic that depends on the local clock |
| Reorg handling | Rollback of state, indexes, the mempool and wallet balances during deep reorgs |
| **Supply accounting for shielded pools** | Do pools with hidden amounts have independent inflow and outflow accounting (such as Zcash's turnstile) that limits losses from forgery when the proof system fails? Is there continuous monitoring of the total supply? |
| Light clients and proofs | Are the header chain, Merkle proofs and signatures that light clients rely on fully validated? |
| State and storage | Consistency of the state tree, pruning and snapshot recovery; can corrupted data leave a node stuck in a wrong state? |
| **Verification result caches** | When the results of verifying signatures, proofs and scripts are cached, does the key cover the full verification context with an unambiguous encoding (variable-length fields need a length prefix)? Can a cache hit skip conditions that should be checked again? |

## 4.46 Federated identity and single sign-on

Applies to: code that uses OAuth 2, OpenID Connect or SAML as a client, resource server or issuer; services that accept challenge-response authentication such as NTLM and Kerberos; and code that issues certificates or tickets for accounts. For general checks on sessions and tokens, see [4.5](#45-authentication-sessions-and-tokens); for signature verification, see [4.4](#44-cryptography-and-credentials).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Redirect URIs** | Does the authorization server compare the redirect URI character by character against the full pre-registered address? Wildcard, prefix or subpath matching, or an open redirect on an allowed domain, all send the authorization code or token somewhere else |
| **state, PKCE and authorization codes** | Does the authorization request carry a state or PKCE bound to the current session? When the request included `code_challenge`, is a token exchange without `code_verifier` rejected? Is the `plain` method rejected? Can an authorization code be used only once, and is it bound to the client and the redirect URI? When it is reused, are the tokens already issued revoked? |
| Multiple identity providers | When working with several authorization servers, is the issuer (`iss`) in the response checked, or are redirect URIs kept separate per provider, so that one provider's authorization code cannot be handed to another? |
| **ID tokens and access tokens** | Are ID tokens checked for issuer, audience equal to this client, validity period and nonce? Can a token issued to another client be used to log in to this app? Does the resource server check that the access token's audience is itself? |
| Account lookup and linking | Are local accounts located by issuer plus subject identifier (`iss` + `sub`)? Does the "link a third-party account" flow have cross-site protection, or can an attacker link their own external account to the victim's account? |
| Deprecated flows | Is the implicit flow or the password flow still used? Do tokens appear in URL fragments, browser history or the Referer? |
| Client types and credentials | Do public clients such as mobile apps and single-page apps carry a client secret? Do confidential clients authenticate at the token endpoint in a way that resists replay? |
| **SAML assertions** | Is a valid signature required on the exact assertion the application actually reads, with unsigned ones rejected? Are audience, recipient, destination, validity period and `InResponseTo` all checked? Are assertion IDs recorded to prevent replay? |
| **Consistent XML parsing** | Do signature verification and attribute reading use the same parser and the same parse result? Are DOCTYPE, comments, namespaces and duplicate elements handled the same way in both parses (see also [4.24](#424-webhooks-and-external-events) ("What is verified"))? |
| **Relay of challenge-response authentication** | Do services that accept NTLM or similar challenge-response authentication require signing, or bind authentication to the TLS channel and the target service name (Extended Protection)? Otherwise, authentication that someone is tricked into starting can be passed on to log in to this service |
| Identity source when issuing credentials | When issuing certificates, tickets or tokens, is the subject name taken from attributes the requester can change themselves (host name, display name, email)? When a lookup by name finds nothing, does it fall back to variants such as adding a suffix or dropping the domain, and land on a different subject? Are all trusted fields covered by the signature? |

## 4.47 Local privileged components and local privilege escalation

Applies to: code that runs with higher privileges than its callers and also accepts requests from other users or processes on the same machine, or handles files they can reach. Examples include Windows services, macOS privileged helpers and XPC services, Linux setuid programs, D-Bus and polkit services, installers and updaters, and agents and security software with system privileges. For how widely a local control port is open, see [4.35](#435-client-apps-extensions-and-auto-update); for environment inheritance of child processes, see [4.22](#422-subprocesses-dynamic-execution-and-decoder-side-effects); for system calls themselves, see [4.30](#430-os-interfaces-system-calls-and-descriptors).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Caller identity** | How does the server side of a named pipe, local socket, XPC, D-Bus or ALPC identify its caller: by credentials the kernel attaches to the message (audit token, `SO_PEERCRED`, code signing requirements), or by a process ID, a self-reported name or a path? Getting the process ID first and then looking it up is a race, and process IDs get reused |
| **File operations in low-privilege writable locations** | When a privileged process creates, moves, deletes or changes permissions of files in user-writable directories (temporary directories, logs, user configuration, shared data directories), can any level of the path be swapped for a system file midway through a symbolic link, directory junction, hard link or opportunistic lock? |
| **Executable paths and load order** | The program path of a service or scheduled task contains spaces but is not quoted; the file or directory of a program, script, plugin or library that runs with privileges is writable by low-privilege users; the library search order looks first in the current directory, the application directory or writable directories in `PATH` |
| Caller-controlled environment | Does the privileged component locate its own security database, helper programs and logs using environment variables the caller controls (`HOME`, `TMPDIR`, `PATH`, locale settings) or configuration the user can change? |
| Injection that borrows granted permissions | Do programs that hold system privacy grants, special entitlements or the ability to escalate privileges allow loading unsigned libraries or having code injected (dynamic library environment variables, disabled library validation, debugging entitlements, Electron's `RunAsNode` switch)? If they do, any program on the machine can borrow their permissions |
| Impersonating the caller | After a service impersonates the client's identity, does every exit path restore its own identity? When the service acts as a client and connects to a pipe or path that a low-privilege user can create, does it forbid the other side from impersonating it? |
| Dropping privileges | Is the order of dropping privileges correct (supplementary groups first, then the group, and the user last)? Is the return value of each step checked? After dropping, is it confirmed that the original privileges cannot be regained? |
| Install, repair and uninstall | Do install, repair and update flows that run with high privileges load files or run scripts from user-writable directories? After uninstalling, are privileged helpers, services, scheduled tasks and drivers all removed too? |

## 4.48 Kernel drivers, device emulation and virtualization

Applies to: kernel modules and drivers (including drivers for GPUs, displays, storage and security software), and low-level code that handles requests from less-trusted parties with high privileges, such as hypervisors, device emulation and paravirtualized backends. For general issues with system calls and descriptors, see [4.30](#430-os-interfaces-system-calls-and-descriptors); for memory safety, see [16](dimensions.md#16-language-and-runtime-pitfalls).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Access control on device entry points** | Can ordinary users open the device node or device object? Is each control code (IOCTL) checked separately against the caller's privileges, instead of "if you can open it, you can call it"? |
| **Dangerous primitives** | Does it give user mode the ability to read and write arbitrary physical memory, kernel addresses, MSRs or I/O ports, or to map arbitrary memory ranges? Once such a driver is signed and released, others will carry it to other machines to escalate privileges |
| User pointers and lengths | Are pointers, lengths and offsets from user mode validated before access (range, alignment, landing at the start of an object) and copied into the kernel once? Is direct access to user buffers probed first? Is the same data read twice (see [4.30](#430-os-interfaces-system-calls-and-descriptors) ("Reading untrusted memory twice"))? |
| Kernel information leaks | Are structures copied to user mode zeroed first (padding bytes, arrays that are not fully written)? Do error paths and logs leak kernel addresses? |
| **Mapping lifetime and permissions** | After kernel, device or GPU memory is mapped into user mode, are the mappings revoked in step when pages are freed, migrated or change permissions? Can read-only pages be written through a mapping on the device side? |
| Objects and concurrency | When control code handling runs concurrently with close, free or device removal, do reference counts and locks cover it? Can user mode stretch the race window with tricks such as multiple threads or page faults? |
| **Guest-controlled device emulation** | Register values, DMA descriptors and ring queue indices of virtual devices are all controlled by the guest: are they validated and read only once before use? Do legacy devices that are enabled by default but not actually needed widen the attack surface? |
| Signing and block lists | When a released driver turns out to be vulnerable, is its signature revoked, or is it submitted to the system's vulnerable driver block list? Does the party that loads third-party drivers enforce the block list? |

## 4.49 Mobile app components and inter-process communication

Applies to: source code of Android and iOS apps, including native code, cross-platform frameworks and embedded web content. When only the app package is available, see [Targets that are not source code: Mobile app packages](targets.md#mobile-app-packages); for general client checks, see [4.35](#435-client-apps-extensions-and-auto-update).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Exported components** | Are exported activities, services, broadcast receivers and content providers protected by permissions, or do they check the caller in code? Components that run as the system or with high privileges can be called by any app |
| **Intent redirection** | A received Intent carries another Intent, a component name or a URI inside it, which is used as is to start a component or grant access, letting external apps use this app's identity to open unexported components or read private files |
| Pending intents | Does a PendingIntent use an explicit intent, and is it set to immutable? Once a mutable or implicit PendingIntent is handed out, can its target and data be rewritten? |
| Content providers and file sharing | Are the directories that FileProvider exposes too broad (the root directory, the whole data directory)? Do `openFile` and queries block path traversal and SQL injection? Does it trust file names given by the other side? |
| **Deep links and web views** | Can deep link parameters make an in-app WebView load any web page? Are native bridges (`addJavascriptInterface`, `WKScriptMessageHandler`) open only to trusted origins? WebView file access settings |
| Login callbacks | When the authorization callback uses a custom scheme, can other apps register the same scheme and intercept the authorization code? Does it switch to verified App Links or Universal Links, together with PKCE? |
| Local biometrics | Is the biometric result only a true/false callback, or is it bound to a key in the hardware keystore (Android's `CryptoObject`, access control on the iOS keychain)? What happens if the callback is tampered with to report "passed"? |
| Sensitive data spillover | Are tokens and personal data left in the clipboard, background snapshots and screenshots, system backups, logs, keyboard caches or external storage? |
| Overlays and task hijacking | Can users be tricked into tapping sensitive buttons through a transparent overlay? Can task affinity settings let a malicious app impersonate this app's screens? |

## 4.50 Account abstraction and smart contract wallets

Applies to: contract accounts (smart wallets) and their modules, the entry point contracts and paymasters of account abstraction, EIP-7702 delegation target contracts, and code that needs to verify contract account signatures (ERC-1271). For private keys, frontends and the signing flow, see [4.43](#443-wallets-signing-and-off-chain-components); for general contract checks, see [4.38](#438-smart-contracts-and-on-chain-interaction).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Entry point restrictions** | Can execution and validation functions (such as ERC-4337's `validateUserOp` and `execute`) be called only by the trusted entry point contract or by the account itself? Can others call them directly and bypass signature validation? |
| Signature validation results and nonces | When a signature does not match, does it return the failure code the spec requires, or revert outright? Are the valid-after and valid-until times written into the return value and checked? Can nonce keys and sequence numbers be reused? |
| **Counterfactual addresses** | Is the address computed in advance, before the account is deployed, determined by the owner and the initial configuration? Can someone else deploy an account with a different owner at the same address first? |
| **Paymasters** | Does the contract that pays gas for users check what it agreed to sponsor (the call target, amount, validity period, chain ID and entry point contract address must all be in the signature)? Is there a per-user limit? If charging fails after execution, does it end up paying for nothing? Can its deposit be drained? |
| Validation phase restrictions | Does the validation phase use opcodes or storage access that bundlers forbid (ERC-7562), so operations are rejected and the account becomes unusable? |
| Modules and session keys | How much power do installable validation, execution and hook modules have? Who approves installing and uninstalling them? Are session keys limited to specific targets, functions, amounts and time periods? State and approvals left behind after uninstalling |
| **Contract account signatures** | Verifying signatures with only `ecrecover` rejects contract wallets; is the ERC-1271 return value compared exactly with the magic value? Among several contract accounts owned by the same externally owned account, can a signature be replayed from one to another (the account address must be included in the signed content, as in ERC-7739)? How are signatures from accounts that are not yet deployed (ERC-6492) verified? For administrative operations that are deliberately replayed across chains, is the object named by index or by identity (see [4.4](#44-cryptography-and-credentials) ("Authorized operations that refer to objects by position"))? |
| **EIP-7702 delegation initialization** | Setting the delegation and initializing are not done in the same step: do the initialization parameters require a signature from the externally owned account's private key? Otherwise someone else can initialize it first |
| Delegation switching and storage | After an externally owned account switches its delegation target, the old storage is still there, and the new contract reads the old data through its own layout; does the delegation target use namespaced storage (ERC-7201)? |
| Scope of delegation authorizations | A delegation authorization with chain ID 0 is valid on every chain; does the wallet warn that "signing a delegation authorization" means "handing over the whole account"? Does the delegation target contract itself have replay protection, and does it limit call targets and amounts? |
| Relaying sponsored transactions | Can a relayer that pays gas on behalf of an externally owned account be left paying for nothing because the account empties its balance, changes its nonce or revokes the authorization before the transaction lands on chain? Is there a deposit or reputation mechanism? |
