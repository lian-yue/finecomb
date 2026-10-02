# A.20 PowerShell

Applies to: PowerShell scripts, modules and hosted runspaces. Confirm the edition, version, platform and preference variables; POSIX shell rules do not describe this runtime.

| Checkpoint | What counts as a problem |
| --- | --- |
| Error channels | A non-terminating error leaves later writes running; `try`/`catch` is assumed to catch every failure; a native command's exit code is lost before it is checked |
| Pipeline output | Diagnostic expressions join the success stream and change a function's result; collection unrolling changes the zero, one and many-result cases |
| Argument boundaries | External text is evaluated as a script; native argument quoting is assumed identical across editions and platforms |
| Providers and literal paths | A caller's path is expanded as a wildcard or selects an unintended provider; a file-only operation reaches other resources |
| Remoting | A deserialized object is treated as a live local object; credentials, session state or remote failures are silently lost |
| Scope and runspaces | Dot-sourcing, profiles or reused runspaces retain another invocation's variables, functions or identity |
| Security controls | Execution policy is treated as a sandbox or authorization boundary |

Semantic references: [error handling](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_error_handling), [return and pipeline output](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_return), [execution policies](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies). For process boundaries also use [4.22](specialties/4.22-subprocesses-dynamic-execution-and-decoder-side-effects.md).
