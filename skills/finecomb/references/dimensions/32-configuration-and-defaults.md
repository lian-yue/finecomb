# 32 Configuration and defaults

| Checkpoint | What counts as a problem |
| --- | --- |
| Default values | Unreasonable, undocumented, or inconsistent with the docs; is the default configuration safe |
| Zero value vs. unset | The two need to be told apart but are not ("an explicit 0 means off" vs. "not set means use the default") |
| Hard-coded policy | A configurable policy is written into the code. **Those changed in this change, or directly related to it, must be flagged to the caller**; out-of-scope ones are only reported |
| Validation completeness | Some fields are not validated; validation rules do not match the docs / declared tags; combined validation (A must be less than B) is missing |
| Validation timing | Validated at startup, or only blows up when used |
| Merge semantics | Are the rules for picking values when several configs are merged/inherited (take the larger, the smaller, the first) written down clearly |
| **Options that reset each other** | A setter writes a whole field or bit mask that also holds other options, or options applied in another order overwrite each other, so a security option set earlier (or its secure default) is silently cleared |
| Source precedence | Command line / environment variables / files / defaults — which overrides which; is it written down |
| Units | Sizes and counts mixed up; the range of supported unit suffixes |
| Config that affects identity | Which config items take part in the instance identity, so that changing them means a different set of data — is this written down |
| Secret configuration | Does config loading expose secrets or wrongly inherit permissions; credential exposure and lifecycle are all covered in [4.4](../specialties/4.4-cryptography-and-credentials.md) |
| Hot reload | Can it be changed while running; how a change affects in-flight operations and existing connections; can a bad change be rolled back |
