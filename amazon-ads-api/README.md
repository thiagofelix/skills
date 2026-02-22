# amazon-ads-api Skill

Expert guidance for building with the Amazon Ads API. Uses the live merged OpenAPI spec for exact endpoint signatures and schemas.

## Prerequisites

This skill reads a locally cached copy of the merged Amazon Ads OpenAPI JSON. Download it before use:

```bash
curl -L "https://d1y2lf8k3vrkfu.cloudfront.net/openapi/en-us/dest/AmazonAdsAPIALLMerged_prod_3p.json" \
  -o ~/.amazon-ads-openapi.json
```

Refresh this file periodically to stay in sync with the live spec.

## What It Covers

- **Authentication** — OAuth 2.0 access tokens, client ID header, profile scope header
- **All endpoints in the merged spec** — use `references/api-catalog.md`
- **Common patterns** — pagination, rate limits, throttling, error handling

## Reference Files

| File | Contents |
|------|----------|
| [`references/api-catalog.md`](./references/api-catalog.md) | Tag and endpoint catalog derived from the OpenAPI spec |
| [`references/authentication.md`](./references/authentication.md) | Auth flow, headers, common 401/403 causes |
| [`references/common-patterns.md`](./references/common-patterns.md) | Pagination, rate limiting, error handling |
