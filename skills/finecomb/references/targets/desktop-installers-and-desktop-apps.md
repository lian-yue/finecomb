# Desktop installers and desktop apps

Applies to: Windows MSI and EXE installers, macOS DMG and pkg, Linux deb, rpm, AppImage, Snap and Flatpak, and the desktop apps they install (including frameworks with embedded web content such as Electron). Check the executables inside separately per [compiled artifacts and binaries](compiled-artifacts-and-binaries.md); for the update mechanism, see [4.35](../specialties/4.35-client-apps-extensions-and-auto-update.md).

| Checkpoint | What counts as a problem |
| --- | --- |
| Signing and notarization | Windows Authenticode signatures and timestamps; macOS developer signing, notarization and stapling; repository signatures for Linux packages; whether every executable in the package is signed; whether the signing certificate has expired or been revoked |
| **Install scripts and custom actions** | MSI custom actions, pkg `preinstall` and `postinstall`, and deb and rpm maintainer scripts usually run with the highest privileges: what they execute, what they download, and whether it is verified |
| **Privilege escalation through repair and uninstall** | MSI files are cached in a system directory, and ordinary users can trigger a repair; can the actions run as SYSTEM during the repair be hijacked (referencing files that do not exist, writing to directories ordinary users can write, popping up a command window) |
| Install location and permissions | A program run by a high-privilege service is installed in a directory ordinary users can write; Windows service paths without quotes |
| **Library search paths** | The installer loads a same-named DLL from the download directory; macOS `@rpath` or Linux RPATH points to a writable directory |
| **Changes to the system** | Installing root certificates (when the private key ships with the installer, anyone can issue trusted certificates); changing hosts, proxy or firewall rules; installing drivers, services or scheduled tasks; whether they remain after uninstall |
| Sandbox and permission declarations | Snap `classic` or `devmode` confinement; `--filesystem=host` in Flatpak `finish-args`, and session bus access to `org.freedesktop.Flatpak` (which can escape the sandbox); the macOS App Sandbox and hardened runtime entitlements |
| Embedded web frameworks | Electron's Node integration, context isolation and fuse settings (`RunAsNode`, asar integrity checks, loading only from asar); source code and secrets in asar archives |
| Bundled runtimes | Versions and support lifetime of the bundled Java, Python, Node and OpenSSL (see [36](../dimensions/36-supply-chain-and-artifact-integrity.md)) |
