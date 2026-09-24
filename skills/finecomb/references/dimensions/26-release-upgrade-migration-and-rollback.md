# 26 Release, upgrade, migration and rollback

| Checkpoint | What counts as a problem |
| --- | --- |
| Forward compatibility | What happens when an old version reads data written by a new version — an error, a skip, or a **silent misread** |
| Backward compatibility | Old data that the project still needs to read cannot be read, and there is no agreed migration or explicit rejection |
| Version markers | No version field in the format; no explicit handling when an unknown version shows up |
| Migration | Is the migration idempotent; can it resume after crashing midway; can a failed migration go back to the original state; how long the migration takes and what it locks |
| **Release strategy** | Gradual / canary / batched rollout; how long the observation window is; what triggers an automatic rollback; the risk of shipping everything at once |
| **Rollback mechanism** | Is rollback a single command; how long does it take; what happens while old and new versions coexist during the rollback |
| Data after rollback | After upgrading and then rolling back to the old version, is the data still usable; is it a one-way door (if so, write it down and confirm before release) |
| Mixed-version operation | When the release method lets old and new versions run side by side, is it safe for them to share data and talk to each other; are the prerequisites for a switchover with downtime actually in place |
| Feature flags | The default value, scope and cleanup plan of each flag; have combinations of several flags been verified |
| Breaking interface changes | Have callers migrated per the confirmed plan; whether compatibility is needed is decided by the project contract; do not add a transition layer automatically |
| Configuration migration | Does the meaning of old config change in the new version; is direct replacement or a transition strategy consistent with the authorization |
| Dual-write / dual-read period | Check only when this approach has been adopted: how inconsistencies between old and new data are handled, and when the dual track stops; can the dual-write mapping itself keep both sides consistent (see [23](23-transactions-atomicity-and-consistency.md) ("Incremental sync into a copy that is overwritten whole")) |
| Install and uninstall | With what privileges do install, repair, update and uninstall steps run; do they read files from user-writable locations; after uninstall, are services, scheduled tasks, privileged helpers, drivers, certificates, firewall rules and credentials all removed |
