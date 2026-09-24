# API specification files

Applies to: targets that only have API descriptions such as OpenAPI, AsyncAPI, GraphQL schemas, protobuf or gRPC definitions, or Postman collections. For whether the spec matches the implementation, see [43](../dimensions/43-documentation-consistency.md).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Authentication coverage** | Which operations declare no security requirement; a global requirement removed by a single operation with `security: []`, or an array that contains an empty object `{}`, which makes authentication optional; the authentication method itself is unsuitable (such as an API key in query parameters) |
| **Object identifiers** | Object IDs in paths and parameters can be enumerated (such as auto-increment integers); these operations all need object-level authorization (see [28](../dimensions/28-authorization-and-access-control.md)), and the spec itself cannot prove whether it is done |
| Input constraints | Strings, arrays and numbers without `maxLength`, `maxItems`, `maximum` or `pattern`; objects that allow extra fields (`additionalProperties`), which may allow mass assignment; no limits on upload size and type |
| Output exposure | Response models that contain password hashes, tokens, internal IDs or personal information; error response models that carry internal information |
| Server addresses | `http://` addresses, internal addresses and test environment addresses in `servers` |
| Callback addresses | Addresses in `callbacks`, `webhooks` or parameters that the server will send requests to (see [4.21](../specialties/4.21-outbound-requests-and-server-side-request-forgery.md), [4.24](../specialties/4.24-webhooks-and-external-events.md)) |
| Pagination and batching | List endpoints with no page size limit; batch endpoints with no limit on how many items one call can process (see [20](../dimensions/20-resource-bounds-and-backpressure.md)) |
| Versions and deprecation | Operations marked deprecated but still in the spec; differences between the specs of different versions (for shadow APIs, see [4.14](../specialties/4.14-server-request-handling-and-middleware.md)) |
| Secrets in examples | Real tokens and accounts in `example` fields and Postman environment variables |
| GraphQL and gRPC | Query depth and complexity limits cannot be seen in the spec, so mark them "Unverified"; authorization of mutations; whether gRPC reflection is enabled |
