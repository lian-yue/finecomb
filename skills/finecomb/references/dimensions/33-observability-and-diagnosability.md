# 33 Observability and diagnosability

| Checkpoint | What counts as a problem |
| --- | --- |
| Log placement and frequency | Logging on hot paths; flooding the log on failure; logging cannot be turned off or have its level changed |
| Log and error content | Leaking keys, paths, credentials or user data — see [30](30-privacy-data-governance-and-compliance.md) |
| Locatable errors | The operation type, safe resource identifiers and correlation information are missing, so failures cannot be located; the needed context must still meet the data minimization requirements of [30](30-privacy-data-governance-and-compliance.md) |
| Correlation IDs | Can things be tied together across components / requests (request ID, trace ID); are the IDs passed all the way down |
| Exposed statistics | Are the exposed counts/sizes/states **exact or estimates**; are the error sources and convergence conditions written down |
| Estimates used as exact | The docs say it is an estimate, but callers (and tests) use it as exact |
| Key paths visible | Are the few numbers you most want when something goes wrong (queue length, in-flight count, failure rate, latency percentiles) exposed |
| **Service objectives** | Are there explicit availability / latency targets; can they be measured; who finds out when they are missed |
| **Alerts** | Do critical failures have alerts; are the alerts actionable, are they noisy, can they miss things |
| **Ownership and runbooks** | Who owns this; where are the steps for handling problems written; can a newcomer follow them |
| Self-check entry points | Is there a way to run a consistency check; can internal state be verified when something goes wrong |
| On-site information | Is the information you get when something goes wrong enough to reconstruct what happened |
