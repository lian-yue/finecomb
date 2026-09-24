# 9 Resource exhaustion and protocol-level denial of service

| Pattern | Historical cases | What to look for | Where it lands |
| --- | --- | --- | --- |
| **Cheap operations traded for expensive work** | HTTP/2 Rapid Reset (CVE-2023-44487, the client creates streams and cancels them at once, but the server still has to process them) | Whether cheap operations such as cancel, reset and close make this side pay an asymmetric cost; whether work after cancellation still counts toward limits | [20](../dimensions/20-resource-bounds-and-backpressure.md) ("Cost asymmetry"), [4.1](../specialties/4.1-networking-and-connections.md) |
| Hash collision flooding | PHP (CVE-2011-4885) and many other languages at the same time | Untrusted keys entering hash tables that have no random seed | [20](../dimensions/20-resource-bounds-and-backpressure.md) |
| Regex backtracking | Cloudflare's global outage in 2019 (backtracking in one WAF rule) | Backtracking regex engines processing external input or rules | [20](../dimensions/20-resource-bounds-and-backpressure.md), [4.29](../specialties/4.29-rule-and-policy-matching.md) |
| Decompression and entity expansion | "Billion laughs" XML entity expansion; decompression bombs | Whether total decoded output, nesting depth and entry count have limits | [4.3](../specialties/4.3-decoding-untrusted-external-data.md) |
| Reflection amplification | Memcached UDP amplification (CVE-2018-1000115) | Whether a connectionless protocol sends back more data than the request before it verifies the source address | [4.28](../specialties/4.28-tunnels-proxies-and-the-network-data-plane.md) |
| Crashes caused by counter overflow | TCP SACK Panic (CVE-2019-11477) | Whether counts and segment counts that the peer controls overflow in the kernel or protocol stack | [4.2](../specialties/4.2-protocols-and-frame-parsing.md), [14](../dimensions/14-boundaries-numbers-and-text.md) |
