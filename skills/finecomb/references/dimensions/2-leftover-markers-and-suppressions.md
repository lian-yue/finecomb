# 2 Leftover markers and suppressions

> These are not dead code. They are **unfinished items and silenced warnings that others left behind**. Each one stands for a decision or a debt, and the review must judge, one by one, whether each still holds.

| Checkpoint | How to check | What counts as a problem |
| --- | --- | --- |
| TODO / FIXME / XXX / HACK / to-verify | Full-text search | Judge each one: does it still hold? Whose responsibility is it? Should it be done, deleted or written into the docs? Do not assume "leaving it there is fine" |
| Commented-out code | Search for runs of commented-out code lines | Is the reason for stopping written down; has the commented-out content drifted away from the current implementation |
| **Static analysis suppressions** | Search for the suppression markers of lint, type checks and security scans in each language (for the syntax, see the "Leftover markers" row in [Part VII](../tools.md)) | Behind each suppression is a decision or an unfixed issue; does the reason still hold, has it gone stale as the code changed |
| Empty implementations / no-ops | Look for implementations whose body is empty, only returns zero values, only does `pass` or only throws "not implemented" | Is it an intentional empty implementation or unfinished work; an intentional one must say so |
| "Unreachable" crash points | Search for explicit termination and assertions: panic, abort, assert, unreachable, fatal, process exit calls, "should not reach here" exceptions | Is it really unreachable? Can public input reach it? Are the assertions still there in release builds (see [16](16-language-and-runtime-pitfalls.md))? |
| Debug leftovers | Hard-coded switches, always-true conditions, temporary prints (each language's print, console, debug log level), breakpoint statements, hard-coded test addresses and accounts | Slipped into production |
| **Leftover backups and build outputs** | Backup directories, copies and build outputs in the source tree | Should be cleaned up or have a clear reason to stay; they pollute search results and static analysis |
| "For now", "later", "TBD" in documents | Full-text search | Stale plans taken as the current contract |
