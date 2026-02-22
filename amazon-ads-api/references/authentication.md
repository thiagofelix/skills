# Authentication

Amazon Ads API uses OAuth 2.0. Most requests require an access token derived from a refresh token.

## Token Exchange

- Exchange refresh token for access token via the OAuth token endpoint.
- Access tokens expire quickly; cache and refresh proactively.

## Required Headers

- `Authorization: Bearer <access_token>`
- `Amazon-Advertising-API-ClientId: <client_id>`
- `Amazon-Advertising-API-Scope: <profile_id>` (profile scope header)

## Common 401/403 Causes

- Expired or invalid access token
- Missing or incorrect `Amazon-Advertising-API-ClientId`
- Missing or incorrect profile scope header
- Token does not have the required advertising scope

For exact auth requirements, check the OpenAPI `security` sections and the official Ads API docs.
