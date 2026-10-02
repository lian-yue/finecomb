# 23 Transactions, atomicity and consistency

> This dimension asks "**which things must become true together**". Transactions are not only for databases: any code that "changes two places" must answer these questions.

| Checkpoint | What counts as a problem |
| --- | --- |
| Unit of atomicity | Which operations must succeed or fail together; where the boundary is drawn; is it written down |
| Transaction boundaries | Where the transaction opens and where it closes; is there a rollback on error paths; is there a long transaction that wraps slow operations inside it |
| Nesting and reuse | The semantics of nested transactions; the same transaction committed separately by several layers |
| Isolation level | Which guarantees does the selected implementation actually provide; can dirty reads, non-repeatable reads, phantoms, lost updates or write skew violate an invariant spanning several records even when individual writes do not conflict |
| **Consistency promises** | Strong consistency / eventual consistency / read-your-writes / monotonic reads — what is promised, does the implementation match it, where is it written |
| Cross-resource operations | How are operations across two stores / two services / a store plus a cache kept atomic; is there compensation |
| **Incremental sync into a copy that is overwritten whole** | When incremental changes are synced into another copy that is read and written as whole objects (another store, an old format, a cache, a downstream system), is the whole object rebuilt from the complete state after the change; after a deletion, are all remaining entries written back, or only the entries that appear in the increment; is there a periodic full reconciliation as a backstop |
| Compensation and rollback | What if the compensation itself fails; is the compensation idempotent; can the compensation clash with the normal path |
| Dedup and idempotency keys | How repeated requests are recognized; whether a key binds the caller, operation and relevant request content; whether conflicting reuse is rejected; how reservation, completion and stored results survive retries and failure; what happens after expiry |
| **"Exactly once"** | Where exactly-once is claimed, is the real semantics "at least once + an idempotent consumer"; is this stated clearly |
| Visibility order | Is there a promise on the order in which writes become visible to other readers; is the cache written first or the store |
| Read and write paths differ | Writes go one way and reads go another (such as write to the store, read from the cache); how long is the window in between |
| Retrying transactions | After a serialization failure or deadlock, does retry restart the required reads and decisions rather than reuse stale values or a failed transaction; can external effects performed inside the attempt repeat |

Semantic references: [transaction isolation and retry boundaries](https://www.postgresql.org/docs/current/transaction-iso.html), [idempotent request identity](https://docs.aws.amazon.com/ec2/latest/devguide/ec2-api-idempotency.html). Apply the actual store or service contract; these implementations do not define universal defaults.
