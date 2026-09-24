# Device communication and IoT protocols

Applies to: when you only have the device or gateway, captured wireless and messaging traffic (BLE, Zigbee, MQTT, CoAP and so on), or the configuration of these protocols. For firmware, see [firmware and device images](firmware-and-device-images.md).

| Checkpoint | What counts as a problem |
| --- | --- |
| **BLE pairing** | With legacy pairing (LE Legacy Pairing), capturing the pairing is enough to compute the key; "Just Works" pairing does not protect against man-in-the-middle attacks; sensitive characteristics can be read or written without requiring encryption and authentication |
| BLE privacy | The device uses a fixed address instead of a resolvable random address, so it can be tracked |
| **Zigbee network joining** | Still accepts the public default Trust Center link key (`ZigBeeAlliance09`); install codes are not used; the join window stays open too long; the network key is sent encrypted with a publicly known key |
| **MQTT authentication and authorization** | Anonymous connections are allowed; the service is only on plaintext port 1883; topic permissions are not isolated per device, so any client can subscribe to `#`; retained messages contain secrets; a client ID collision can kick another device offline |
| Provisioning | Whether the hotspot used for first-time provisioning is open; whether the Wi-Fi password is sent in plaintext |
| Device identity | Whether each device has its own certificate or key, or they all share one (see the firmware checkpoint "Default credentials and shared keys") |
| Replay and freshness | Can commands such as unlock or on/off be recorded and replayed |
| Over-the-air updates | Whether update packages are signed and verified over the wireless link (see [4.36](../specialties/4.36-embedded-firmware-and-real-time-constraints.md)) |
| Amplification and exposure | UDP-based services such as CoAP and SSDP can be used for reflection amplification when they are exposed externally |
