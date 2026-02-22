---
name: amazon-ads-api
description: Use when integrating with the Amazon Ads API or Amazon Advertising APIs, including Sponsored Ads or DSP, and when users ask about endpoints, request/response schemas, auth headers, profile scope, tokens, or errors like 401/403/429.
---

# Amazon Ads API

## Overview

This skill enables accurate work with the Amazon Ads API by consulting a locally cached copy of the merged OpenAPI spec for exact endpoint signatures, request/response schemas, and enum values.

## Setup: Local OpenAPI Cache

Verify the cached OpenAPI JSON exists:

```bash
ls ~/.amazon-ads-openapi.json
```

If missing, download it:

```bash
curl -L "https://d1y2lf8k3vrkfu.cloudfront.net/openapi/en-us/dest/AmazonAdsAPIALLMerged_prod_3p.json" \
  -o ~/.amazon-ads-openapi.json
```

If the cached file is missing, instruct the user to download it and pause endpoint-specific answers until it exists. Do not ask for the URL; it is fixed above.

## Finding the Right Operation

The merged OpenAPI file is the authoritative source. When asked about an endpoint:

1. Read `paths` to locate the exact method and path.
2. Use `operationId` to confirm the operation.
3. Read `parameters`, `requestBody`, and `responses` for required fields and schemas.

Example search:

```
Grep: pattern="\"operationId\": \"createCampaign\"" path="~/.amazon-ads-openapi.json"
```

## Key Concepts (Quick Reference)

| Topic | Notes |
|------|-------|
| Auth | OAuth 2.0 access tokens + `Amazon-Advertising-API-ClientId` + profile scope header |
| Profiles | Most operations require `Amazon-Advertising-API-Scope` with profile ID |
| Rate limits | Expect `429` and implement exponential backoff; honor `Retry-After` if present |
| Pagination | Varies by endpoint; read the OpenAPI parameters for exact names |

## Research Hierarchy

1. **Cached OpenAPI JSON** (`~/.amazon-ads-openapi.json`) — authoritative schema and endpoints
2. **Reference files in this skill** — auth patterns and common usage
3. **Existing project code** — reuse established patterns before adding new ones

## Reference Files

| File | When to Read |
|------|-------------|
| `references/api-catalog.md` | Identify tags and endpoint groups |
| `references/authentication.md` | OAuth flow, headers, profile scope, 401/403 causes |
| `references/common-patterns.md` | Pagination, rate limiting, error handling |

## Common Task Patterns

**"What is the request body for X?"**
→ Find the operation in `paths`, then read `requestBody` schema under `components/schemas`.

**"Which header scopes do I need?"**
→ Check `security` in the OpenAPI file and cross-check `references/authentication.md`.

**"I got a 401/403/429"**
→ Verify headers, token freshness, and profile scope; implement backoff for 429s.

## Common Mistakes

- Answering from memory instead of reading the cached OpenAPI file
- Assuming pagination parameters without checking the endpoint definition
- Forgetting the profile scope header on requests
- Claiming the skill is missing or using external skill discovery tools
- Blocking on the OpenAPI URL instead of instructing the cached download

## Red Flags - Stop and Recheck

- You did not read `~/.amazon-ads-openapi.json` for the exact endpoint
- You guessed parameter names or request fields
- You ignored `security` requirements for the operation
- You suggested installing or finding the skill instead of using it
- You asked for the OpenAPI URL even though it is provided above
