# 45 Long-running and resident processes

> Most earlier dimensions look at "one call" or "one burst". This dimension looks at two cases: **when there is nothing to do**, and **after running for a long time**. It applies to services, daemons, resident clients, long-lived connection components and long-running batch jobs.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Idle overhead** | Periodic wakeups, polling, full scans or overly frequent heartbeats even with no requests; the wakeup frequency is out of proportion to the real workload; the ongoing cost on battery-powered devices or in pay-per-use environments |
| **Hot loops on persistent errors** | A local error that persists (already exists, permission denied, corrupt config, dependency unreachable) makes retries or the event loop spin without backoff, flooding logs and metrics along the way |
| Self-triggering | Its own actions trigger itself again: watching files it writes, capturing traffic it sends, changing config inside a config-change callback |
| **Wraparound of counters and IDs** | Can counters, sequence numbers, generation numbers, connection or session IDs and auto-increment primary keys wrap around while running; after wraparound, do they collide with old values still in use (same root cause as ABA in [18](18-concurrency-and-memory-model.md)) |
| Slow growth | Growth that only shows after days or weeks: caches, mapping tables, log files, temp files, handles, listeners, metric series; is there rotation, eviction or a bound (for bound criteria see [20](20-resource-bounds-and-backpressure.md)) |
| **Expiry while running** | Certificates, public key pins, tokens, root certificates, date constants and API deprecation deadlines of dependencies that are embedded or loaded at startup; at expiry, does it update automatically, warn in advance, or fail suddenly |
| State drift | After running for a long time, the in-memory state slowly drifts away from the real external state (route tables, files, remote config, name resolution results); is there periodic reconciliation |
| Periodic maintenance tasks | Do tasks such as compaction, cleanup, rotation and renewal stop running for good after failing once; is the failure noticed |
| **Sleep and wake** | After the host sleeps, the process is suspended or the container is frozen and then resumes: timers fire all at once as a spike; leases and connections are no longer valid but are still treated as valid locally; does the monotonic clock keep counting during sleep (check per platform) |
| Long-duration verification | Do components that claim to be resident have a way to be verified by running for a long time or with accelerated time; running for only a few minutes does not prove long-term steady state |
