# Host and client environments

Applies to: the host and client contexts selected in [Appendix B](../environments.md). Read only the selected sections.

## Unix services

Does behavior still hold under the service manager's identity, supplementary groups, umask, resource limits, signal delivery and inherited descriptors? Are Linux-specific calls or `/proc` assumptions used on a different Unix host?

When a service manager passes an already open listener or accepted connection, do its actual address, protocol, permissions and ownership satisfy the checks otherwise performed during local creation?

Check [34](../dimensions/34-runtime-environment-and-deployment-contract.md) and [4.30](../specialties/4.30-os-interfaces-system-calls-and-descriptors.md).

## Windows services and applications

Do drive-relative, UNC and device paths, alternate streams, reparse points, ACLs and file-sharing modes preserve the intended file boundary? Does the actual service token have the permissions assumed from an interactive session?

Check [4.8](../specialties/4.8-file-systems-and-paths.md) and [4.47](../specialties/4.47-local-privileged-components-and-local-privilege-escalation.md). Semantic reference: [Microsoft file and namespace documentation](https://learn.microsoft.com/en-us/windows/win32/fileio/naming-a-file).

## macOS applications and helpers

Do launchd context, sandbox entitlements, privacy permissions, signing and quarantine state allow the intended behavior? Does a helper authenticate the caller independently of the GUI application?

Check [4.35](../specialties/4.35-client-apps-extensions-and-auto-update.md) and [4.47](../specialties/4.47-local-privileged-components-and-local-privilege-escalation.md).

## Android and iOS applications

After permission revocation, background suspension, process death or account switching, can pending callbacks act on stale identity or unavailable storage? Are device-only behavior and native plugins distinguished from emulator evidence?

Check [4.49](../specialties/4.49-mobile-app-components-and-inter-process-communication.md).

## Browser pages and embedded WebViews

Which origin, secure context, sandbox flags and storage partition apply? Does SSR or hydration move secrets or security decisions into client code? Are unsupported browser APIs and revoked permissions handled?

Check [4.18](../specialties/4.18-user-interfaces-and-accessibility.md) and [4.35](../specialties/4.35-client-apps-extensions-and-auto-update.md).

## Workers and service workers

Can the host stop work before side effects finish? Do cache upgrades, offline responses, message channels and old open clients preserve identity and data-version boundaries?

Check [4.54](../specialties/4.54-sync-collaboration-and-offline-clients.md) and [4.35](../specialties/4.35-client-apps-extensions-and-auto-update.md).
