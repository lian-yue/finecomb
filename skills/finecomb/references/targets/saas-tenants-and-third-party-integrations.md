# SaaS tenants and third-party integrations

Applies to: when you are given only a read-only administrator identity for a SaaS platform such as an office suite, code hosting or customer management, or when you need to review third-party apps and integrations connected to these platforms.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Third-party app authorization** | The permission scopes held by authorized OAuth apps and integrations (read and write all mail, all files, all repositories); whether ordinary users can grant authorization on their own; whether the publisher is verified; long-unused authorizations that have not been revoked |
| **Long-lived tokens and keys** | Personal access tokens, API keys and service account keys with no expiry or not rotated for a long time; integration-only accounts with too many permissions |
| **Login and MFA** | Is single sign-on enforced; is MFA enforced for administrators and all users; are local accounts that bypass single sign-on and legacy authentication protocols (such as IMAP and POP with basic authentication) turned off |
| Administrators | The number of super administrators; how emergency accounts are kept and monitored; whether test tenants or old tenants are connected to production |
| External sharing | "Anyone with the link" sharing; guests and external collaborators; public repositories, boards and forms |
| Data exfiltration channels | Rules that auto-forward to external mailboxes; destinations of webhooks and data export connectors (see [4.24](../specialties/4.24-webhooks-and-external-events.md)) |
| Audit logs | Are they enabled, and how long are they kept; are key events not recorded because the license tier is too low |
| Offboarding and revocation | Are departed employees' accounts, tokens and authorizations revoked together with the identity source |
| Code hosting platforms | Organization default permissions; branch protection or rulesets; permissions of the default CI token; deploy keys; secret scanning and push protection (see [4.33](../specialties/4.33-build-scripts-ci-and-infrastructure-as-code.md)) |
