---
name: amazon-sp-api
description: Expert guidance for building with the Amazon Selling Partner API (SP-API). Use this skill whenever the user is working on Amazon seller integrations, SP-API endpoints, order management, inventory, listings, fulfillment, reports, finances, or any Amazon marketplace automation. Triggers on mentions of SP-API, Selling Partner API, Amazon seller API, MWS migration, amazon orders/inventory/listings code, LWA tokens, marketplace IDs, or any of the 50+ SP-API domains. Always use this skill when the user asks about specific Amazon API endpoints, request/response shapes, or integration errors.
---

# Amazon Selling Partner API

## Overview

This skill enables precise, accurate work with the Amazon Selling Partner API by consulting local OpenAPI model files for exact endpoint signatures, request/response schemas, and operation details. It covers all 51+ SP-API domains from orders and inventory to vendor APIs and the Data Kiosk.

## Setup: Local Models Repository

Before starting any SP-API work, verify the models repository exists locally:

```bash
ls ~/.sp-api-models/models
```

If missing, clone it:

```bash
git clone https://github.com/amzn/selling-partner-api-models ~/.sp-api-models
```

This repository contains OpenAPI (JSON) specs for every SP-API endpoint. Having it local means you can look up exact request parameters, response shapes, enum values, and required vs optional fields — no guessing.

## Finding the Right Model File

Once the repo is cloned, use Glob to find the relevant spec:

```
Glob: ~/.sp-api-models/models/<api-folder>/*.json
```

Or search by operation name across all models:

```
Grep: pattern="getOrders" path="~/.sp-api-models/models" glob="*.json"
```

See `references/api-catalog.md` for the complete list of all 51 API domains and their folder names.

## Working with Model Files

SP-API models are OpenAPI 3.0 JSON files. When a user asks about a specific endpoint:

1. **Find the right folder** using the catalog in `references/api-catalog.md`
2. **Read the JSON spec** — focus on `paths` (endpoints) and `components/schemas` (request/response types)
3. **Check operation parameters** — distinguish required vs optional, note enum constraints
4. **Verify the response schema** — understand what fields are guaranteed vs conditional

Example — to look up the `getOrders` operation:
```
Read: ~/.sp-api-models/models/orders-api-model/ordersV0.json
```
Then find `"operationId": "getOrders"` for the exact parameters and response shape.

## Key Concepts (Quick Reference)

**Authentication**: SP-API uses Login with Amazon (LWA) — OAuth 2.0. Most operations need a seller's refresh token exchanged for an access token. Some operations are "grantless" (use client credentials only). Restricted operations (PII data) require a Restricted Data Token (RDT). See `references/authentication.md` for the full flow.

**Sandboxes**: Every SP-API endpoint has a sandbox variant at `sandbox.sellingpartnerapi.com`. Sandbox responses are predefined — check the model's `x-amazon-spds-sandbox-behaviors` extension for the exact canned responses available.

**Rate Limiting**: Each operation has a burst rate and restore rate (token bucket). When throttled you get a `429` — always implement exponential backoff. The `x-amzn-RateLimit-Limit` response header tells you the current rate.

**Pagination**: Most list operations use `nextToken` (a cursor). Pass it back in the next request. Some newer APIs use `pageToken`. Always check the specific model's parameter names.

**Marketplace IDs**: Operations are scoped to a marketplace. Common ones: `ATVPDKIKX0DER` (US), `A1F83G8C2ARO7P` (UK), `A1PA6795UKMFR9` (DE), `APJ6JRA9NG5V4` (IT), `A1RKKUPIHCS9HS` (ES). See `references/common-patterns.md` for the full table.

## Research Hierarchy

When answering SP-API questions, consult sources in this order:

1. **Local model files** (`~/.sp-api-models/`) — authoritative source for exact schema details
2. **Reference files in this skill** — auth patterns, common patterns, API catalog
3. **Existing code in the user's project** — check for established patterns before introducing new ones

## Reference Files

| File | When to Read |
|------|-------------|
| `references/api-catalog.md` | Identifying which model folder to use, understanding API scope |
| `references/authentication.md` | LWA tokens, RDT, grantless operations, OAuth flow |
| `references/common-patterns.md` | Pagination, error handling, rate limiting, sandboxes, marketplace IDs |

## Common Task Patterns

**"How do I get orders?"**
→ Read `~/.sp-api-models/models/orders-api-model/ordersV0.json`, find the `getOrders` operation

**"What fields does a listing need?"**
→ Glob `~/.sp-api-models/models/listings-items-api-model/*.json`, read `ListingsItemPutRequest` schema

**"How do I create an inbound shipment to FBA?"**
→ Read `references/api-catalog.md`, then read `~/.sp-api-models/models/fulfillment-inbound-api-model/`

**"How do I subscribe to order notifications?"**
→ Read `~/.sp-api-models/models/notifications-api-model/notifications.json` for `createSubscription` + `references/authentication.md` for destination setup

**"I'm getting a 403"**
→ Read `references/authentication.md` — most 403s are expired tokens, wrong scopes, or missing RDT for restricted data operations

**"What reports are available?"**
→ Glob `~/.sp-api-models/models/reports-api-model/*.json` and check the `reportType` enum values
