# Browser extensions

Applies to: extension packages (CRX, XPI, ZIP) for browsers such as Chrome, Edge, Firefox and Safari, or the version listed in a store. For the code level, see [4.35](../specialties/4.35-client-apps-extensions-and-auto-update.md).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Permissions and site scope** | `permissions`, `host_permissions` and content script `matches` go beyond what the features need; high privileges such as `<all_urls>`, `cookies`, `webRequest`, `debugger`, `scripting` and `nativeMessaging` |
| **Remote code** | Downloading and running remote scripts, `eval`, running remote configuration as code; store policies forbid this, but sideloaded and old versions may still have it |
| **Communication with web pages** | Which websites `externally_connectable` allows to send messages; whether messages received by `onMessageExternal` and `postMessage` messages received by content scripts are checked for origin and structure; whether a web page can drive the extension's privileged interfaces |
| Resources web pages can access | Pages exposed by `web_accessible_resources` can be embedded by web pages (clickjacking), or used to detect which extensions the user has installed |
| Content security policy | The content security policy of extension pages is relaxed |
| Secrets in local storage | Tokens stored in `storage.local`, or in `storage.sync`, which syncs to the account; whether the web page context can read them |
| Native program communication | `allowed_origins` in the native messaging host manifest; whether the host program validates input |
| **Updates and publishing accounts** | A custom `update_url`; whether the store publishing account has multi-factor authentication (MFA) enabled and who can publish. In late 2024, several extensions pushed malicious updates after their developers were phished |
| Data sent out | Where collected browsing history, page content and form data are sent (see [30](../dimensions/30-privacy-data-governance-and-compliance.md)) |
