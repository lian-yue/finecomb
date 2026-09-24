# 24 Encoding and persistent formats

| Checkpoint | What counts as a problem |
| --- | --- |
| Self-description | Are the existing format markers, schema or external constraints enough to identify the data; foreign or unsupported formats get processed as valid data |
| Integrity checks | No length or checksum, so truncation cannot be detected; does the check cover all of the content |
| Reserved fields | Not zeroed on write; not checked on read |
| Byte order | Inconsistent from place to place |
| Untrusted input | See [4.3](../specialties/4.3-decoding-untrusted-external-data.md) |
| Identity stability | For the key → storage location mapping, can a config change make it hit old data; is the order of the inputs that produce the mapping stable (see [15](15-algorithm-and-data-structure-correctness.md)) |
| Foreign data | Can someone else's data in the same location be wrongly deleted, read or counted |
| Local corruption | Can one bad record keep the whole thing from opening — can it skip it and go on |
| **Round trip and canonical form** | Decoding and then re-encoding is not equivalent to the original data; unknown fields are silently dropped, or passed through as-is without the contract saying so; when one value has several encodings, a non-canonical form is accepted while signatures, hashes, dedup or cache keys depend on the canonical form |
| Version compatibility | See [26](26-release-upgrade-migration-and-rollback.md) |
