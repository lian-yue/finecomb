# Databases and datasets

Applies to: data exports, read-only connections or data files.

| Checkpoint | What counts as a problem |
| --- | --- |
| Access permissions | Permissions of accounts and roles; who can read sensitive tables |
| Sensitive data inventory | Which tables and fields hold personal information, credentials and payment data; whether they are encrypted or masked |
| **How passwords and tokens are stored** | The password hash format in the user table (`$argon2id$`, `$2b$`, or unsalted MD5 or SHA-1 hex strings); tokens, API keys and security question answers stored in plaintext (criteria in [4.5](../specialties/4.5-authentication-sessions-and-tokens.md)) |
| Database configuration | Listen address, transport encryption, default or empty-password accounts, authentication not enabled, audit switches; check this when you have a read-only connection, and record it as "Not checked" when you only have an export file |
| Tenant isolation | Is multi-tenant data isolated by tenant column and row-level security; views and functions that run with definer privileges |
| Constraints and integrity | Do the expected unique, foreign key, not-null and check constraints exist; does the data already contain records that break business rules |
| Backup and retention | Are backups encrypted, and are they regularly verified by restoring; is expired data cleaned up |
| The export file itself | Where this export is stored, whether it is encrypted, who can get it; treat it as sensitive data too |
