# Common Patterns

## Pagination

Pagination varies by endpoint. Always read the OpenAPI spec for the exact parameter names and response fields. Common patterns include:

- Index-based paging (`startIndex` + `count`)
- Token-based paging (`nextToken`)

## Rate Limiting and Retries

- Expect `429` responses when throttled.
- Use exponential backoff with jitter.
- Honor `Retry-After` if present.

## Error Handling

- Parse error responses and log `requestId` or trace identifiers when present.
- For `401/403`, verify auth headers and profile scope.

## Reporting Workflows

Some endpoints are asynchronous (request -> poll -> download). Use the OpenAPI spec to identify polling status fields and download URLs.
