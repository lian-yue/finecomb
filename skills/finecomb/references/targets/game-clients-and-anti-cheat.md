# Game clients and anti-cheat

Applies to: game clients, anti-cheat components and their drivers, and online game protocols that can only be observed from the client. Do this on a test server or private server, not on live servers: testing there affects other players and violates the terms of service.

| Checkpoint | What counts as a problem |
| --- | --- |
| **The client has the final say** | Position, damage, currency, items and cooldowns are computed and reported by the client, and the server trusts them directly |
| Protocol | Messages have no integrity protection and no sequence numbers; they can be replayed, sped up or reordered; the server does not check rates and physical limits |
| In-app purchases and rewards | Receipts are not verified with the platform on the server side; reward delivery is not idempotent (see [4.39](../specialties/4.39-payments-accounting-and-billing.md)) |
| **Anti-cheat drivers** | Control interfaces exposed by the kernel driver do not check the caller; a signed driver is taken by others as a "bring your own vulnerable driver" to disable security software (ransomware abused mhyprot2.sys in 2022); whether it remains after uninstall |
| Privacy and permissions | The scope of the process, file and hardware information collected by anti-cheat, and where it is sent (see [30](../dimensions/30-privacy-data-governance-and-compliance.md)) |
| Anti-debugging and obfuscation | They only raise the cost and are not a security boundary; they are no reason to skip server-side validation |
| Mods and scripts | The execution capabilities granted when loading user mods, scripts and maps (see [4.27](../specialties/4.27-interpreters-compilers-and-virtual-machines.md)) |
