# Model files and datasets

| Checkpoint | What counts as a problem |
| --- | --- |
| File format and loading | Does the model file use a format that runs code at load time (such as pickle-based formats); a different format is not automatically safe: does the loader import and run code based on configuration, class names, function names, templates or a remote-code switch in the file (see [4.22](../specialties/4.22-subprocesses-dynamic-execution-and-decoder-side-effects.md)); are a safer format and safer loading options available |
| Source and integrity | Can the source, digest and license be verified |
| Data issues | Personal information in the data; sources that may be poisoned |
| Claims versus measurement | Are the capabilities and limits claimed in the model description backed by evaluations |
