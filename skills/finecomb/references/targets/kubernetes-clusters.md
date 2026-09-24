# Kubernetes clusters

Applies to: Kubernetes clusters viewed with a read-only identity, whether self-hosted or managed. The control plane of a managed cluster is run by the vendor, so the related baseline items are recorded as not applicable, and that vendor's CIS benchmark is used instead. When you only have manifest files, check per [configuration, deployment manifests and infrastructure code](configuration-deployment-manifests-and-infrastructure-code.md).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Overly broad RBAC** | Subjects bound to `cluster-admin`; wildcard verbs and resources; `escalate`, `bind` and `impersonate`; subjects that can create Pods, read Secrets or run `exec` effectively hold the node or its credentials |
| **Unauthenticated entry points** | The API server allows anonymous access; the kubelet is unauthenticated or its read-only port is open; the dashboard or etcd is reachable from outside or does not use certificate authentication |
| Workload isolation | The Pod security level of each namespace (privileged, baseline, restricted); privileged containers, host directory mounts, host network or process namespaces, extra capabilities, running as root |
| **Workloads submitted by tenants** | When the platform runs Pods, jobs or containers for tenants, are spec fields let through only by an allowlist (run-as UID, shared process namespace, volumes, capabilities, host network); is an allowed UID the same as that of a platform component such as a service mesh sidecar, which lets the workload bypass that component's traffic rules; can the workload share a process namespace with a sidecar the platform injects, and read that sidecar's token and configuration |
| Service account tokens | Tokens are auto-mounted by default; long-lived token Secrets; workloads use the default service account |
| **Network isolation** | No default-deny network policy; the network plugin in use does not enforce network policies, so written policies have no effect; Pods can reach the cloud metadata service |
| Secret protection | Encryption at rest is not enabled for Secrets; secrets placed in ConfigMaps or environment variables; who can list Secrets |
| Admission and images | Is there an admission policy; are images pinned to digests, only from trusted registries, and signature-verified |
| Audit logs | Is the API server audit policy enabled; its log level and retention |
| Versions and support lifetime | Are the cluster and node versions still within the support period of upstream or the managed service vendor |
