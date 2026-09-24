# Cloud accounts and infrastructure state

Applies to: cloud accounts or platform configuration viewed with a read-only identity. For clusters, also see [Kubernetes clusters](kubernetes-clusters.md).

| Checkpoint | What counts as a problem |
| --- | --- |
| Identity and permissions | Overly broad roles and policies, keys unused for a long time, administrators without MFA, cross-account trust |
| Public exposure | Public storage buckets, snapshots and images; security groups and load balancers open to the whole internet |
| **Instance metadata service** | Is the version that requires a session token enforced (such as AWS IMDSv2); once a workload is hit by SSRF, can it get cloud credentials (see [4.21](../specialties/4.21-outbound-requests-and-server-side-request-forgery.md)) |
| **Federated identity trust conditions** | Do roles that trust external identities (such as OIDC tokens from CI) restrict the repository, branch and audience; when only the issuer is trusted and the subject is not restricted, other people's workflows can also assume the role |
| Cross-account trust | Do cross-account roles given to third parties require an external ID, to prevent misuse (confused deputy) |
| Root and emergency accounts | Does the root account have access keys; does it have MFA; has it been used recently |
| Organization-level guardrails | Organization policies, bans on public sharing, region restrictions; when a read-only identity cannot see higher-level policies, record them as "Not checked" |
| Secrets in managed services | Secrets in the environment variables of function and container services; permissions of function execution roles; public function URLs |
| Logs and monitoring | Is audit logging enabled, how long is it retained, can it be tampered with |
| Encryption and keys | Encryption at rest, key rotation, who can use the keys |
| Drift from the declared state | Drift between the actual state and the infrastructure code (see [4.33](../specialties/4.33-build-scripts-ci-and-infrastructure-as-code.md)) |
