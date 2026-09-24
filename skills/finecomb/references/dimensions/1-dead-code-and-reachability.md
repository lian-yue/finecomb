# 1 Dead code and reachability

| Checkpoint | How to check | What counts as a problem |
| --- | --- | --- |
| Unused declarations | Cross-check static analysis results with the call graph, registration mechanisms, build conditions and external APIs | Has no real use in any supported scenario; a public API cannot be judged dead only because nothing in the project references it |
| **Production code used only by tests** | First confirm what the tool can do, then confirm with production and test callers | Serves only tests and has no production duty; the difference between the tool's output in its two modes is only a lead |
| Unreachable branches | Follow config constraints and callers backward to find the real value domain of each branch | Branches that upstream validation already rules out (e.g. a parameter is validated as positive, but the code still keeps a whole path for it being ≤ 0) |
| Leftovers of old implementations | Look for aliases, wrappers, forwarders, old paths, deprecation markers, comments like "used to be called X" | A new implementation has replaced the old one, but the old one is still there |
| Speculative design | Look for parameters nobody passes, fields nobody reads, switches not wired up, interfaces with only one implementation and no need for substitution | "In case we need it later" |
| Dead tests | Unused helpers, cases that only cover deleted paths | See [41](41-test-quality.md) |

**Decision rule**: classify only after confirming the real use, the reachability conditions and any explicit intent to keep it. Whether to delete it, keep it commented out or change platform code follows the project rules; a read-only review only reports the basis for the classification.
