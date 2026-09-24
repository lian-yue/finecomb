# Mobile app packages

Applies to: installation packages such as APK, AAB and IPA. The full criteria for components and inter-process communication are in [4.49](../specialties/4.49-mobile-app-components-and-inter-process-communication.md); this section only lists what to check first when you have the package.

| Checkpoint | What counts as a problem |
| --- | --- |
| Manifest and configuration | Debuggable, backups allowed, cleartext traffic allowed, network security configuration, iOS ATS exceptions |
| **Exported components and deep links** | Exported activities, services, broadcast receivers and content providers; parameter validation for URL schemes and universal links (see [4.49](../specialties/4.49-mobile-app-components-and-inter-process-communication.md) ("Deep links and web views"), then [4.35](../specialties/4.35-client-apps-extensions-and-auto-update.md)) |
| **Embedded secrets** | API keys, client secrets, private service addresses; secrets that belong on the server placed in the client |
| Certificate validation and pinning | Whether certificate validation is skipped; whether certificate pinning is done, and what happens when the pinned certificate expires or is rotated |
| Local storage | Where tokens and personal data are stored (the system keychain or Keystore, or plaintext files, preferences or databases); screenshots and the clipboard |
| **Local authentication** | Biometrics that only return a boolean before letting the user in can be bypassed by hooking; the result should be bound to a key in the system keystore (Android `CryptoObject`, iOS keychain access control) |
| Platform interaction | Mutable `PendingIntent`; forwarding a received Intent as is; overly broad `FileProvider` path configuration (such as `root-path`); sensitive screens without protection against overlays |
| Target OS version | A low `targetSdkVersion` keeps old insecure defaults (below 24, user-installed certificates are trusted; below 31, components with intent filters are exported by default); a low minimum version lacks system protections |
| Embedded web content | JavaScript interfaces, file access and mixed content exposed by WebView |
| **Cloud backend configuration** | Firebase, object storage and identity pool configuration in the package; whether database rules or storage buckets are publicly readable or writable. Verifying this means sending requests to the service, so handle it according to the authorization |
| Dynamic loading | Whether code or script bundles downloaded and loaded at runtime (hot updates) have their signatures verified |
| Native libraries | Check the `.so` files and frameworks in the package for hardening and versions per [compiled artifacts and binaries](compiled-artifacts-and-binaries.md) |
| Third-party SDKs | What data analytics, advertising and crash-reporting SDKs collect and send out (see [30](../dimensions/30-privacy-data-governance-and-compliance.md)) |
| Privacy declarations | Whether the iOS privacy manifest (`PrivacyInfo.xcprivacy`), the store's data safety declaration and what third-party SDKs actually collect agree with each other |
| Signing and obfuscation | Signing scheme and certificate; whether security depends on "secrecy" from obfuscation |
| Forced updates | After a vulnerability is found, can old versions be disabled or forced to upgrade |
