# 11 Usability and misuse resistance

> A library's "usability" is not about how nice the interface looks. It is about **how hard it is for others to use it wrong**. A clearly written contract does not mean it is hard to misuse.

| Checkpoint | What counts as a problem |
| --- | --- |
| Least surprise | Is the behavior consistent with the name and with the conventions of similar interfaces |
| **Hard to misuse** | Are there usages that are "easy to get wrong yet still compile"; can types, scopes or required parameters stop wrong usage at compile time |
| Dangerous defaults | Can a caller trigger dangerous behavior beyond what they expect without knowing it; for concrete default values see [32](32-configuration-and-defaults.md) |
| Confusable parameters | Adjacent parameters of the same type are easy to swap (two strings, two durations, two identifiers) |
| Calls that must be paired | Acquire/release, begin/end, lock/unlock — what happens if one half is missed; can a scope mechanism force the pairing |
| Partial initialization | Can a "half-built" object be constructed and used |
| Silent failure | Used wrong but with no feedback at all (returns a zero value, ignores a parameter, silently degrades) |
| Actionable error messages | After misuse, does the error point straight at how to fix it |
| Examples are the contract | Are the doc examples the recommended usage; does copying an example walk into a trap |
| Escape hatches | When a high-level wrapper blocks low-level capabilities, is there a way out; is there a risk the way out gets abused |
