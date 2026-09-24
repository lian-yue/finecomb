# A.10 Shell

| Checkpoint | What counts as a problem |
| --- | --- |
| **Unquoted expansions** | `$var` without quotes goes through word splitting and glob expansion |
| Not stopping on errors | No `set -e`; `set -e` has no effect in conditions, pipelines, `local x=$(cmd)` and similar places; `set -u` and `pipefail` are missing |
| **Dangerous deletion** | When the variable is empty, `rm -rf "$DIR/"` becomes `rm -rf "/"`; block this with `${DIR:?}` |
| Arguments taken as options | File names that start with `-` are taken as options; separate them with `--` |
| Injection | `eval`; putting external input into a command, or into a command string that runs remotely |
| Temporary files | Predictable names (such as `/tmp/foo.$$`); use `mktemp` |
| Changing directory | The script keeps running in the wrong directory after `cd` fails |
| Reading input | `read` without `-r`; `IFS` has been changed |
| Subshells | A `while` in a pipeline runs in a subshell, so variables changed inside it are lost once it ends |
| Portability | bash features used under `/bin/sh`; old bash shipped with the system; argument differences between BSD and GNU tools (such as `sed -i`) |
| PATH | Relying on `PATH` to find commands while the caller can control the environment (see [4.22](specialties/4.22-subprocesses-dynamic-execution-and-decoder-side-effects.md)) |
