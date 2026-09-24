# 25 Crash and recovery

Tell the actual faults apart: normal shutdown, exceptions, graceful termination, forced kill, power loss, out of memory and storage failure. Forced kill and power loss cannot rely on process cleanup; whether cleanup can still run when a container stops or storage turns read-only must be checked against the real signals and error paths. Recovery guarantees follow the fault scope the project promises.

| Checkpoint | How to check | What counts as a problem |
| --- | --- | --- |
| Classifying termination modes | For each termination mode, ask "which line can still run this time" | Only normal shutdown is considered; recovery relies on things that only the normal path writes |
| **Durability** | When durability across power loss is promised, check the file, directory entry and commit syncs the target platform needs, and how sync failures are handled | A cached write or an atomic replace is treated as a durable commit; data can still be lost on power loss after success is returned |
| Enumerating crash points | Along the write path, ask **line by line**: "if power is lost here, what is the state outside the process? Can the next open tell?" | An intermediate state exists that cannot be determined |
| Write order | Log first, then data? Can the order support recovery? Can the lower layer reorder it | If the order assumption does not hold, the whole recovery fails |
| Partial writes | A write may only get halfway (short write); torn writes at sector/page granularity | Relying on the assumption "one write is atomic"; not checking the byte count a write returns |
| Publication atomicity | Writing the final state directly vs. a temp file plus an atomic replace; is the cost of each written down | The cost of writing directly (a failed overwrite loses the old data) is not in the docs |
| Recognizing half-finished data | Use write ownership, commit markers, length and integrity checks to tell in-flight writes apart from leftover fragments | Time or length alone cannot prove the content is fully committed; recovery wrongly deletes data that is still being written, or accepts damaged data of the same length |
| Recovery coverage | Does recovery cover every intermediate state, or only the few that someone thought of | Holes |
| **Crash during recovery** | Recovery crashes again halfway through | Recovery is not idempotent → the second open is worse |
| Idempotent replay | Does replaying the log twice give the same result | Not idempotent |
| Recovery cost | How much must be scanned after a restart; how the time grows with the data volume | At the scale it claims to support, startup needs a full scan |
| External interference | Someone deleted the directory / changed the data / the mount point disappeared / it became read-only | Crashes outright, or silently loses data |
| Resource exhaustion | Disk full, inodes used up, handles used up, system calls interrupted | Not handled, or after handling, the in-memory state and the persistent state disagree |
| **Multiple processes** | Check the transactions, conditional writes or lock mechanism for shared writes, and how they are released after a crash | Without effective coordination, processes overwrite each other, or leftover locks block recovery; do not call it an issue just because there is no file lock |
| In-process exclusivity | The same process opens the same instance twice | Two sets of state overwrite each other |
| Backups and drills | Are there backups; **has the recovery flow been drilled**; are the backups themselves usable | Backups exist but have never been verified to restore |
