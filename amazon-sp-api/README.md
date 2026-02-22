# amazon-sp-api Skill

Expert guidance for building with the [Amazon Selling Partner API](https://developer-docs.amazon.com/sp-api/). Covers all 51+ SP-API domains — orders, inventory, listings, fulfillment, reports, finances, vendor APIs, and more.

## Prerequisites

This skill reads local OpenAPI model files for exact endpoint signatures and schemas. Clone the official models repo before using it:

```bash
git clone https://github.com/amzn/selling-partner-api-models ~/.sp-api-models
```

## What It Covers

- **Authentication** — LWA token exchange, grantless operations, Restricted Data Tokens (RDT) for PII
- **All 51+ API domains** — see [`references/api-catalog.md`](./references/api-catalog.md) for the full list
- **Common patterns** — pagination (`nextToken`), rate limiting with exponential backoff, error handling, sandbox testing
- **Async workflows** — reports (request → poll → download) and feeds (upload → submit → poll)
- **Listings** — PUT vs PATCH semantics, product type schema validation

## Reference Files

| File | Contents |
|------|----------|
| [`references/api-catalog.md`](./references/api-catalog.md) | All 51 API domains and their model folder names |
| [`references/authentication.md`](./references/authentication.md) | LWA OAuth flow, RDT creation, common 403 causes, token refresh strategy |
| [`references/common-patterns.md`](./references/common-patterns.md) | Pagination, rate limiting, error codes, sandbox setup, marketplace IDs, reports/feeds/notifications patterns |

## How the Skill Works

When invoked, the agent:
1. Looks up the relevant OpenAPI spec in `~/.sp-api-models/models/`
2. Reads exact request parameters, response shapes, and enum values from the JSON
3. Consults the reference files for auth patterns and common patterns
4. Checks existing project code before introducing new patterns

This eliminates guessing on parameter names, required vs. optional fields, and response schemas.
