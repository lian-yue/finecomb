# 35 Dependencies

| Checkpoint | What counts as a problem |
| --- | --- |
| Necessity | The standard library or the project already has an equivalent capability |
| Usage | A whole library pulled in for one small function |
| Understanding the contract | Are the assumptions about the dependency's behavior guaranteed by its docs; does the code rely on its undefined behavior; when the dependency calls back into this code, and this side replaces or changes the context, config or collection the dependency is using inside the callback, does the dependency's contract allow that |
| Failure modes | Behavior when the dependency fails / gets slow — see [22](22-fault-isolation-and-degradation.md) |
| Upstream forks | Is the boundary still "upstream mirror + minimal local patches"; are the patches still needed, is there a public alternative; does the sync method still work |
| Transitive dependencies | Size, licenses and conflicting versions of transitive dependencies |

> The trustworthiness of dependencies, lock file integrity and vulnerability scanning are checked in [36 Supply chain and artifact integrity](36-supply-chain-and-artifact-integrity.md).
