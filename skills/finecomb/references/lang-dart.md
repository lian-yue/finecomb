# A.22 Dart and Flutter

Applies to: Dart libraries and Flutter applications, including native, JavaScript and WebAssembly builds. Select the deployed backend rather than assuming development VM behavior.

| Checkpoint | What counts as a problem |
| --- | --- |
| Null and dynamic input | `dynamic`, casts, `!` or uninitialized `late` fields let external input trigger runtime failures despite static null safety |
| Futures and zones | A Future's failure is unobserved or reaches a different error zone; a timeout is treated as cancellation of the underlying operation |
| Streams | Subscriptions survive their owner; pause buffers grow without bounds; code assumes broadcast streams replay missed events |
| Isolates | Message transfer is confused with shared memory; mutable state or native resources are assumed available in another isolate; failures and exit messages are ignored |
| UI lifecycle | After an `await`, a widget or context is no longer mounted, but the callback updates it or acts on a replaced account |
| Backend differences | Numeric precision, native libraries, reflection or isolate support is assumed identical in native and web builds |
| Native plugins | Platform-channel data is trusted without validation; native callbacks outlive the Dart owner; platform-specific permissions are missing |

Semantic references: [Dart concurrency](https://dart.dev/language/concurrency), [Flutter web deployment and build modes](https://docs.flutter.dev/deployment/web). Also select the matching [environment](environments.md) and [mobile components](specialties/4.49-mobile-app-components-and-inter-process-communication.md) when applicable.
