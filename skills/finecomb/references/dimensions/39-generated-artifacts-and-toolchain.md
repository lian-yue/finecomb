# 39 Generated artifacts and toolchain

| Checkpoint | What counts as a problem |
| --- | --- |
| Freshness of generated artifacts | Generated artifacts lag behind the generation source |
| Hand-edited artifacts | Someone edited a generated file directly, and it will be overwritten |
| Reproducible generation | The generator version is not pinned, so different machines produce different artifacts |
| **Maintenance boundaries** | Are the generation source, generated artifacts, upstream mirrors and local patches kept apart; writes, docs and verification of affected paths all follow the project rules; do not mechanically require scanning or testing the whole upstream code |
| The generator itself | When the target contains generators, macros or build-time plugins, review them per [4.31](../specialties/4.31-code-generators-and-build-time-tools.md) |
