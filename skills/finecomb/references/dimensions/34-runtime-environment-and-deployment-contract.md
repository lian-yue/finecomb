# 34 Runtime environment and deployment contract

| Checkpoint | What counts as a problem |
| --- | --- |
| Environment assumptions | Are the assumptions about directory existence, permissions, network reachability, time zone, locale and temp space written down; when they are not met, is there a clear error or a strange failure |
| Environment variables | When they are read (at startup vs. on each use), behavior when missing, default values, case and naming conventions |
| **Awareness of resource limits** | Are concurrency, memory pools and buffer sizes computed from the actual quota or from the host's total — computing from the total inside a container badly oversubscribes. Whether the runtime is aware of container limits varies by language and version; see [Appendix A](../languages.md) |
| Container specifics | Signal forwarding and zombie reaping when running as PID 1; read-only root file system; where the writable directories are; the size of the temp directory |
| Handles and ports | Where the limits come from; what happens when they are exceeded |
| Startup dependencies | When a dependency is not ready, does it retry and wait or fail right away; is there an implicit assumption about startup order |
| Shutdown contract | How long it is allowed to take after a termination signal; it gets force-killed on timeout; does that duration match the internal shutdown timeout |
| Clock and localization | Does it depend on the host time zone / locale; can behavior differ across hosts |
| Single-instance assumption | What happens when several replicas run at the same time; is there an implicit "I am the only one" assumption (local locks, local caches, local counts) |
| Storage assumptions | Is the local disk persistent; is it still there after a restart; do replicas share the same storage |
| Runtime identity and isolation | Does it use system or cloud permissions beyond its duties; do container privileges, host mounts, runtime sockets, service accounts and secret mounts widen the impact after a compromise |
| Network and management plane | Do the actual listen addresses, management interfaces, debug interfaces, object storage public policies and ingress rules match the deployment contract; do not assume that being "on the internal network" means no identity or permission checks are needed |
| Configuration drift | Do the artifacts actually deployed and the config actually in effect match the review input; do environment overrides, side entry points or security config that is not enabled change the conclusion |
