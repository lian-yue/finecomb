# 22 Fault isolation and degradation

> The previous dimension asks "can I hold up on my own". This one asks "**when others break, what do I do**".

| Checkpoint | What counts as a problem |
| --- | --- |
| Behavior when a dependency fails | What happens when each external dependency fails: hard failure, degradation, default values, stale cache — is it defined |
| Circuit breaker | Is there a circuit breaker; the transition conditions of its state machine (closed / open / half-open), the recovery conditions, the probe volume while half-open |
| Bulkhead isolation | Can one class of requests using up resources drag down another; are pools, queues and execution flows separated by class |
| **Timeout budget allocation** | How the total budget is split across stages; is the downstream timeout smaller than the upstream's remaining time; can the timeouts of serial hops add up to more than the total budget |
| **Retry storms** | Do retries at several layers multiply the calls; are budget, backoff and jitter effective; can side effects that already happened or whose result is unknown be retried safely; for the idempotency criteria see [23](23-transactions-atomicity-and-consistency.md) |
| Observable degradation | Is anyone told when degradation happens; can you see whether it is in a degraded state right now |
| Automatic recovery | Does it recover by itself after the dependency recovers; is manual intervention needed; does recovery cause a thundering herd |
| Partial availability | When some features break, do the others still work, or is everything down |
| Slow is worse than down | What happens when a dependency gets slow (not failing) — can timeouts catch it; do connections / execution flows get used up |
| Cache as degradation | When degrading to stale data, how stale is too stale to use; is the caller told the data is stale |
