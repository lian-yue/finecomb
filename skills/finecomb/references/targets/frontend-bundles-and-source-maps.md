# Frontend bundles and source maps

Applies to: when you only have the built web frontend (bundled scripts, styles, HTML), source maps, or the live assets of a single-page application.

| Checkpoint | What counts as a problem |
| --- | --- |
| Restoring source | When `sourcesContent` in the source map can restore the original source, review it as source code; what is restored is only the frontend |
| **Secrets in the bundle** | Keys, private API addresses and server-side keys for third-party services built into the frontend; the frontend should only contain identifiers that can be public |
| Hidden routes and features | Admin page routes, feature flags, calls to unreleased APIs; they reveal the list of server APIs, and authorization still has to be checked on the server (see [28](../dimensions/28-authorization-and-access-control.md)) |
| Client-side authorization | Permissions controlled only by hiding buttons or routes |
| Third-party scripts | Source and integrity checks of external scripts (see [4.18](../specialties/4.18-user-interfaces-and-accessibility.md)) |
| Dangerous DOM writes | Places such as `innerHTML`, `dangerouslySetInnerHTML` and `eval` that receive external data (see [4.18](../specialties/4.18-user-interfaces-and-accessibility.md)) |
| Dependency versions | Libraries bundled in and their versions (check per [36](../dimensions/36-supply-chain-and-artifact-integrity.md)); bundling often strips version information, so note that conclusions from fingerprinting are inferences |
| Debug leftovers | Debug logs, test accounts and mock APIs in the production bundle |
