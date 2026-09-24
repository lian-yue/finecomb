# 29 Fail-safe behavior and dangerous operations

> On error, does the system stop on the **safe side** or on the dangerous side. This dimension matters most for code that "changes external state".

| Checkpoint | What counts as a problem |
| --- | --- |
| **Failure direction** | On error, deny or allow by default; when the auth / rate limiting / validation component is down, does it block everything or let everything through |
| Confirming dangerous operations | Do irreversible operations such as delete, overwrite, migrate and reset have a second confirmation / preview / dry run |
| Irreversibility | Which operations cannot be undone, and is that written down; is there soft delete or a retention period as a safety net |
| **Limiting the scope of impact** | Do dangerous batch operations have a scope limit; **does it refuse to run when there is no limiting condition** (e.g. a delete with no filter) |
| Dry runs and rehearsals | Is there a mode that only reports and does not execute; can the rehearsal result differ from the real run |
| Operation preconditions | Are there constraints such as "must not run under these conditions" (no backup, version mismatch, not enough space, unexpected target); are they checked |
| Risk warnings | Do dangerous paths give explicit warnings in the docs, names and logs |
| Blast radius | How much can one mistake destroy at most; is there batching, rate limiting, a circuit breaker |
| Non-idempotent destructive retries | What happens when a non-idempotent destructive operation is retried |
| Recovery path | Is there a way back after a mistake; has the way back been verified |
| **Emergency controls** | A leaked credential cannot be rotated or revoked at once without an outage; a compromised component, feature or integration cannot be switched off quickly; the switch itself is not protected or not logged |
