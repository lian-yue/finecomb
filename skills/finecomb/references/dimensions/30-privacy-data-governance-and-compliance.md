# 30 Privacy, data governance and compliance

| Checkpoint | What counts as a problem |
| --- | --- |
| Data classification | Which fields are personal / sensitive information; are they labeled |
| Minimization | Is what is collected and stored actually needed; is there any "keep an extra copy while we are at it" |
| Retention period | How long data is kept; is it deleted when it expires; where the period comes from |
| **Whether deletion is real** | After deletion, is it still in backups, logs, caches, indexes, derived data or message queues; is soft delete treated as real deletion; does old content stay in a file overwritten in place without truncation, or does removed or redacted content remain in an edited output (see [4.8](../specialties/4.8-file-systems-and-paths.md) ("Overwriting in place")) |
| Anonymization | Is anonymization / pseudonymization reversible; can people be re-identified through linked fields |
| Cross-border transfer and region | Region limits on where data is stored and where it is sent |
| Logs and telemetry | Is there personal information or credentials in logs, metric labels, traces or crash reports |
| Purpose limitation | Does the use go beyond the scope declared when the data was collected |
| Third-party sharing | Does data passed to third parties (including libraries, SDKs and external services) go beyond what is needed |
| Encryption requirements | Are the encryption requirements for data in transit and at rest met |
| Source of requirements | Have the applicable regions, data categories, contracts or organizational policies been confirmed; when the source and version are unclear, record the requirement as pending confirmation, and do not write technical check results up as legal compliance certification |
| Data subject rights and consent | Can people's requests to view, export, correct or delete their own data be completed, and do they cover every copy; does consent record its scope and time, and does the related processing stop after consent is withdrawn |
| Real data in tests and samples | Is there real personal data or credentials in test fixtures, samples, screenshots, recorded responses or log samples |

For the completeness and retention policy of access records for sensitive data, see [4.23](../specialties/4.23-security-audit-logs.md).
