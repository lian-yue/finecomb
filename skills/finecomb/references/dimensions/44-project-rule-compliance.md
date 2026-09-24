# 44 Project rule compliance

Check item by item against **the target project's own rules**. Cover at least these categories; the specific clauses follow that project's rules files:

| Category | What to check |
| --- | --- |
| Directories and responsibilities | Is new content placed where its responsibility belongs; were other modules changed out of bounds |
| Change scope | Does it go beyond the scope authorized this time; are out-of-scope issues only reported and left untouched |
| Maintenance boundaries | Does it respect the generation and upstream boundaries in [39](39-generated-artifacts-and-toolchain.md), and the platform verification limits in [38](38-portability-and-build-context.md) |
| Naming and style | Naming rules, comment rules, formatting rules |
| Compatibility layers | When an old implementation is replaced, are the old names fully removed |
| Tool and command boundaries | Do commands, reads, writes and external access follow the project rules and this authorization; the examples in this checklist cannot stand in for authorization |
| Caches and temp directories | Do their location and naming follow the rules |
