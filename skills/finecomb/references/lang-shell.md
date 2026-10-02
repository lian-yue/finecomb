# A.10 Shell

Applies to: POSIX-style shells and their dialects. Use [A.20](lang-powershell.md) for PowerShell; its pipelines and error handling follow different rules.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Unquoted expansions** | `$var` without quotes goes through word splitting and glob expansion |
| Not stopping on errors | Required failures are lost in conditionals, pipelines, substitutions or declarations, so dependent operations continue; verify explicit status handling and the selected shell's `errexit`, `nounset` and `pipefail` behavior rather than treating missing switches alone as defects |
| **Dangerous deletion** | When the variable is empty, `rm -rf "$DIR/"` becomes `rm -rf "/"`; block this with `${DIR:?}` |
| Arguments taken as options | File names that start with `-` are taken as options; separate them with `--` |
| Injection | `eval`; putting external input into a command, or into a command string that runs remotely |
| Temporary files | Predictable names (such as `/tmp/foo.$$`); use `mktemp` |
| Changing directory | The script keeps running in the wrong directory after `cd` fails |
| Reading input | `read` without `-r`; `IFS` has been changed |
| Subshells | A `while` in a pipeline runs in a subshell, so variables changed inside it are lost once it ends |
| Portability | bash features used under `/bin/sh`; old bash shipped with the system; argument differences between BSD and GNU tools (such as `sed -i`) |
| PATH | Relying on `PATH` to find commands while the caller can control the environment (see [4.22](specialties/4.22-subprocesses-dynamic-execution-and-decoder-side-effects.md)) |
| Traps and asynchronous children | Traps lose the original exit status, run cleanup twice or leave background descendants running; `wait` results and pipeline failures are not associated with the operation that owns them |
