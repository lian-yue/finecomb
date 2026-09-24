# 37 Interoperability and coexistence

> One is "can it talk to others", the other is "can it get along with others on the same machine".

| Checkpoint | What counts as a problem |
| --- | --- |
| Standards conformance | Do the formats, protocols and encodings exchanged with the outside conform to public standards; are deviations recorded with reasons |
| Version negotiation | Version / capability negotiation with the peer; behavior on a mismatch; can a downgrade in the negotiation be forced |
| Liberal in, strict out | Tolerant of data received, strict about data sent — is the balance right; can it turn into hiding problems |
| Interop with other implementations | Has it only been tested against its own implementation, or also verified against third-party implementations |
| **Resource coexistence** | When it shares resources (ports, directories, lock files, caches, temp space, shared memory) with other processes / instances on the same machine, can they interfere with each other |
| **Namespace collisions** | Can global names (environment variable names, file names, metric names, registration keys, lock names) collide with someone else's |
| Side effects spilling over | Can changing shared system state (network config, system settings, global singletons) affect others; is it restored on exit |
| Multiple versions side by side | What happens when several versions of the same dependency, or of this program, are present at the same time |
| Yielding resources | Is there a timeout and a way to yield when holding shared resources; can it hold them exclusively for a long time |
