# Design, specification and architecture documents

Applies to: targets that have no code yet, or when only the design is reviewed.

| Checkpoint | What counts as a problem |
| --- | --- |
| Threat modeling | Are data flows, trust boundaries, assets and attackers drawn clearly (see [threat model and attack chains](../baseline.md#threat-model-and-attack-chains)) |
| Failure and abuse scenarios | Do the requirements include error paths, abuse scenarios and limits |
| Turning dimensions into questions | Use the [root-cause facets](../facets.md), the 46 dimensions and the relevant specialties as questions about the design; note that the conclusions are at the design level and the implementation still needs to be checked |
| Verifiability | Can the key security and correctness requirements be tested and audited |
