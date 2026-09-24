# Signatures and provenance

Applies to: any artifact that has a signature, provenance, an in-toto attestation or a transparency log entry, including binaries, installers, images, language packages and firmware.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Verification bound to identity** | Only checks that "there is a valid signature", without checking that the signer is the expected one (certificate subject, OIDC issuer, key fingerprint). When keyless signing does not restrict the identity, anyone can produce a "valid" signature |
| **Attestation matches the artifact** | The subject digest in the attestation differs from the digest of the artifact in hand; the two are matched only by file name or tag |
| **Source and builder** | The source repository, branch or tag, build workflow or build platform in the attestation differs from what is expected; builds from a forked repository, an unprotected branch or a local machine |
| Build level | The claimed SLSA build level has no evidence: L1 only requires provenance to exist; L2 requires it to be signed by a hosted build platform; L3 also requires builds to be isolated from each other and the signing key to be out of reach of build steps |
| Transparency log | Are signatures and attestations recorded in a transparency log (such as Rekor); is the inclusion proof included for offline verification |
| Certificates and time | Was the certificate valid at signing time; is there a trusted timestamp; how are old signatures handled after key rotation and revocation |
| Build inputs | Are the build inputs listed in the attestation (dependencies, base images, tools) pinned to digests |
| in-toto layout | When there is a layout, do the performer of each step and the chain of artifacts between steps all pass verification; who signs the layout itself |
