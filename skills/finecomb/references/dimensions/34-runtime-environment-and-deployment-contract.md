# 34 Runtime environment and deployment contract

Applies to: the hosts and deployment forms the target actually uses. Select their additional questions through [Appendix B](../environments.md); one language can have several deployment contracts.

| Checkpoint | What counts as a problem |
| --- | --- |
| Environment assumptions | Are the assumptions about directory existence, permissions, network reachability, time zone, locale and temp space written down; when they are not met, is there a clear error or a strange failure |
| Environment variables | When they are read (at startup vs. on each use), behavior when missing, default values, case and naming conventions |
| **Awareness of resource limits** | Are concurrency, memory pools and buffer sizes computed from the actual quota or from the host's total — computing from the total inside a container badly oversubscribes. Whether the runtime is aware of container limits varies by language and version; see [Appendix A](../languages.md) |
| Quota enforcement and accounting | Which layer enforces each limit, and what does it count: managed heap, native allocation, mapped or shared memory, child processes, temporary storage and device memory? Can a soft target be exceeded, an ancestor impose a tighter bound, or a quota change invalidate startup sizing? |
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
| Suspend, resume and snapshots | After suspension or snapshot restore, are connections, deadlines, leases, identity and random-generator state still valid; can cloned instances repeat supposedly unique values or ownership |
| Rotated mounts and credentials | Does replacement of a mounted file, credential or trust store take effect in cached handles and clients; can a mixed old/new configuration be observed |
| Runtime capabilities | When a required API, instruction, permission or sandbox capability is absent or revoked, is the failure explicit; does a fallback silently weaken the intended boundary |
| Resolver, proxy and trust configuration | Do effective resolver rules, proxy variables, certificate stores and runtime crypto providers match the intended destination and peer checks; do deployment overrides change them |

Quota reference: [cgroup hierarchy and resource accounting](https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v2.html). Other hosts require their own accounting and enforcement contracts.
