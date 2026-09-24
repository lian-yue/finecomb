# 9 Business logic and flow integrity

> This dimension looks for issues where **"each step is right on its own, but together they can be bypassed"**. Every technical check passes, yet the business rule is still broken through.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Step order** | Can a multi-step flow skip steps, run out of order or repeat; does the server enforce the order, or does it rely only on the UI not letting you click |
| State preconditions | Is each step's requirement on the prior state checked on the authoritative side |
| Timing and races | What happens when the same flow runs twice at the same time (double claiming, over-issuing, double spending) |
| Quantities and quotas | Negative, zero and huge quantities; atomicity of quota deduction and restoration — see [23](23-transactions-atomicity-and-consistency.md) |
| Pricing and amounts | Is the amount passed in by the requester or computed on the authoritative side; stacked discounts, rounding, currency |
| Replay | What happens when the same request is replayed; is there a one-time token or an idempotency key |
| **Anti-automation** | Brute force, bulk enumeration, scalping (bots grabbing limited stock), scraping — is there rate limiting, a challenge, anomaly detection; by what dimension are limits applied (account / address / device / global); where the counters are kept, see [20](20-resource-bounds-and-backpressure.md) ("Storage of limit state") |
| Enumeration leaks | Can resources or accounts be enumerated through differences in responses (exists or not, fast or slow, error wording) |
| Single authority for rules | Are business rules written separately in many places, or is there one implementation; when two places disagree, which one wins |
| Reverse operations | Do the undo, refund and rollback paths have equally strong checks |
| Time windows | Can validity periods, cooldowns and deadlines be bypassed (changing the client time, sitting right on the boundary) |
| **Commit before reveal** | In draws, bidding, matching, guessing and game outcomes, a participant can submit, change, withdraw, abort or retry its choice after seeing the random result, the opponent's choice, the price or the final result; choices should be committed or locked first, and results revealed after; before anything is revealed, is the commitment itself checked to be binding (valid, accepted, impossible to replace) |
| **Conservation of totals** | Does every path that can create value (money, points, inventory, quotas, tokens, credit limits) check conservation: input equals output plus fees, and nothing appears out of thin air; is there a total reconciliation, independent of the main logic, as a safety net; when the design hides the details (shielded pools, aggregated ledgers), what detects forgery |
