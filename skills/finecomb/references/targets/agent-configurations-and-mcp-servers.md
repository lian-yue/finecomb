# Agent configurations and MCP servers

Applies to: when you only have an agent's system prompt, rule files (such as `AGENTS.md`, `CLAUDE.md`, `.cursor/rules/`), skill and tool lists, MCP client configuration (such as `.mcp.json`, `.vscode/mcp.json`, `claude_desktop_config.json`) or the tools declared by an MCP server, and are not reviewing their implementation code. For the code level, see [4.26](../specialties/4.26-llms-and-tool-calling.md).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Secrets in configuration** | Tokens and keys hard-coded in `env`, `headers` or command arguments; configuration files committed to the repository or synced to the cloud |
| **Server source and pinning** | Servers launched with `npx -y`, `uvx` or `docker run` without a pinned version or digest, so every start may run different code; whether the package name impersonates another; who operates the remote server |
| **Tool description poisoning** | Instructions for the model hidden in tool names, descriptions or parameter descriptions; hidden Unicode characters (zero-width, bidirectional controls, tag characters); descriptions quietly changed after the user approved them |
| Name shadowing | Several servers provide tools with the same or similar names; one server's description changes how another tool is used |
| **Permissions and auto-approval** | The list of tools that need no confirmation, allowed command wildcards (for example, allowing any shell), the range of readable and writable directories; whether high-risk tools such as running commands, writing files, sending messages and making payments still require human confirmation |
| **Dangerous combinations** | When the same session can read private data, touch untrusted content and also send data out, external content can carry the data out; judge by combinations of tools, not by single tools only |
| Remote server authentication | Does the remote server require authentication; does the server pass the client's token through unchanged to downstream services; are OAuth scopes minimal |
| Local server exposure | Does a local HTTP server bind only to the loopback address, check `Host` and `Origin` and require authentication; otherwise a web page can call it through DNS rebinding (same-origin requests may carry no `Origin`, so the defense against rebinding relies mainly on checking `Host`) |
| Security carried by prompts | Security rules written only in the prompt and not enforced at the tool execution layer; keys and internal addresses in the prompt (see [4.26](../specialties/4.26-llms-and-tool-calling.md)) |
| Writable context | Who can write long-term memory, retrieval stores, rule files and skill directories; can rule files submitted by others change the agent's behavior |
| Call records | Are tool calls and approvals recorded; can you trace who approved what |
