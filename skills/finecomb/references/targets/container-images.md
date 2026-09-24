# Container images

| Checkpoint | What counts as a problem |
| --- | --- |
| **Secrets in layer history** | A secret added in one layer and deleted in a later layer still remains in the image; secrets in build arguments and environment variables |
| Base image | The base image's source, whether it is pinned to a digest, whether it is past its support lifetime |
| OS package and dependency vulnerabilities | Known vulnerabilities in the OS packages and language dependencies in the image |
| Runtime identity | Whether the default user is root; files with SUID; unneeded tools (shell, package manager, compiler) |
| Entry point and ports | Entry command, exposed ports, health checks |
| Signatures and provenance | Can the image signature and build provenance be verified; is the signer the expected one (see [signatures and provenance](signatures-and-provenance.md)) |
