# Configuration, deployment manifests and infrastructure code

Applies to: targets that only have configuration files, deployment manifests, Helm charts, Kustomize, infrastructure code, or Terraform state and plan files. Check per [32](../dimensions/32-configuration-and-defaults.md), [34](../dimensions/34-runtime-environment-and-deployment-contract.md) and [4.33](../specialties/4.33-build-scripts-ci-and-infrastructure-as-code.md).

| Checkpoint | What counts as a problem |
| --- | --- |
| Secrets | Plaintext credentials in configuration |
| Exposure and privileges | Listen addresses, public ports, privileges, host mounts |
| Defaults and environment differences | Differences from the production environment; dangerous defaults that are not overridden |
| **Check after rendering** | Helm charts and Kustomize overlays must be rendered with the actual values files before checking; looking only at templates and defaults misses conditional branches, and the defaults themselves are often insecure |
| Chart origin | Does the chart repository use HTTPS; is there a provenance file (`.prov`) or signature, and has it been verified; are the dependent subcharts version-locked |
| Install hooks and cluster reads | Permissions of Helm hook jobs; `lookup` in templates reads Secrets from the cluster and returns empty during rendering, so the rendered output differs from the actual install |
| **State and plan files** | Terraform state files and plan files store all resource attributes in plaintext, including database passwords and generated private keys; `sensitive` only affects display |
| State backend | Is state storage encrypted; are versioning and locking enabled; who can read and write it; is the storage bucket public |
| Freshness of state | State is only a snapshot from the last apply or refresh and may already differ from the real environment (see [cloud accounts](cloud-accounts-and-infrastructure-state.md) ("Drift from the declared state")) |
