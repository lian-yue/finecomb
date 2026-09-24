# 4 Query injection

| Pattern | Historical cases | What to look for | Where it lands |
| --- | --- | --- | --- |
| **Concatenated queries off the main path** | MOVEit Transfer (CVE-2023-34362, SQL injection led to mass data theft) | Hand-concatenated queries in file transfer, reporting, export and admin interfaces; sort and field name parameters | [4.7](../specialties/4.7-databases-and-queries.md) |
| Filter conditions decided by the request | Label Studio (CVE-2023-47117, task filters could reach sensitive fields of the user table, guessing other users' password hashes and tokens step by step) | Whether the fields and relation paths that filter, sort and search interfaces allow have an allowlist | [4.7](../specialties/4.7-databases-and-queries.md) ("External objects used as query conditions") |
