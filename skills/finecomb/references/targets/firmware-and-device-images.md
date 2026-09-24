# Firmware and device images

| Checkpoint | What counts as a problem |
| --- | --- |
| Extraction and file system | Can it be unpacked; services, startup scripts and configuration in the file system |
| **Default credentials and shared keys** | Default account passwords; all devices share the same private key, certificate or SSH host key |
| Outdated components | Versions and known vulnerabilities of built-in open-source components (such as busybox, OpenSSL, web servers) |
| **Management interface** | Command injection, path traversal and unauthenticated interfaces in the device's web management pages; this is the most common kind of flaw behind mass exploitation of network devices |
| **Remote services enabled by default** | Whether remote management services such as Telnet, UPnP and TR-069 are on by default, and whether they listen on the WAN port |
| **Secure boot chain** | Whether the bootloader verifies the signatures of the kernel and file system; U-Boot environment variables are writable, and the boot countdown can be interrupted to get a command line; the verification logic is visible in the image, but whether the chip's fuses are blown has to be checked on the physical device |
| Firmware download and encryption | Whether the firmware download address uses HTTPS and the download is verified; encryption is not signing, and firmware that is only encrypted, without signature verification, can still be replaced |
| Update mechanism | Signature verification and anti-downgrade for update packages (see [4.36](../specialties/4.36-embedded-firmware-and-real-time-constraints.md)) |
| Program hardening | Programs in firmware usually lack PIE and stack protection; check per [compiled artifacts and binaries](compiled-artifacts-and-binaries.md) |
| Cloud connection credentials | Whether the keys, certificates and messaging service accounts a device uses to connect to the cloud are unique per device (see [device communication and IoT protocols](device-communication-and-iot-protocols.md)) |
| Debug interfaces | Whether serial ports, JTAG and debug services are turned off in production builds |
| Limits of emulation | The interface running under emulation differs from the real device; parts that fail to run under emulation can only be looked at statically; note this in the conclusions |
