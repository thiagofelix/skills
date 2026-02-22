# SP-API Authentication

## Overview

SP-API uses **Login with Amazon (LWA)** — an OAuth 2.0 implementation. Every API call requires an `x-amz-access-token` header. How you get that token depends on the operation type.

## Three Authorization Types

### 1. Seller-Delegated (Most Operations)

The seller authorizes your application. You get a **refresh token** at authorization time and exchange it for short-lived access tokens.

**Token exchange**:
```
POST https://api.amazon.com/auth/o2/token
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token
&refresh_token=<seller's refresh token>
&client_id=<your app client ID>
&client_secret=<your app client secret>
```

Response: `{ "access_token": "...", "expires_in": 3600 }`

Cache the access token for up to `expires_in` seconds (typically 1 hour).

### 2. Grantless Operations

Some operations don't require seller delegation — they use your application's own credentials. Examples: `getAuthorizationCode`, `createDestination` (notifications), `getAccount` (seller wallet).

**Token exchange**:
```
POST https://api.amazon.com/auth/o2/token
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
&client_id=<your app client ID>
&client_secret=<your app client secret>
&scope=sellingpartnerapi::notifications
```

Available scopes: `sellingpartnerapi::notifications`, `sellingpartnerapi::migration`, `sellingpartnerapi::client_credential:rotation`

### 3. Restricted Data Tokens (RDT)

Some operations return **Personally Identifiable Information (PII)** — buyer names, addresses, phone numbers. These require a separate Restricted Data Token in addition to the regular access token.

**Operations that need RDT**: `getOrders` (with buyerInfo), `getOrder`, `getOrderItems` (with buyerInfo), `getOrderBuyerInfo`, `getOrderAddress`, `getOrderItemsBuyerInfo`, and fulfillment outbound operations with address data.

**Create a RDT**:
```
POST /tokens/2021-03-01/restrictedDataToken
Authorization: <regular access token>

{
  "restrictedResources": [
    {
      "method": "GET",
      "path": "/orders/v0/orders/{orderId}/address",
      "dataElements": ["buyerInfo", "shippingAddress"]
    }
  ]
}
```

Response: `{ "restrictedDataToken": "...", "expiresIn": 3600 }`

Use the RDT as the `x-amz-access-token` for the restricted call. RDTs are single-use scoped — one RDT per resource path pattern.

## Making API Calls

Every SP-API request needs:

```
GET https://sellingpartnerapi-na.amazon.com/orders/v0/orders
x-amz-access-token: <access token or RDT>
x-amz-date: <ISO 8601 datetime>
```

**Regional endpoints**:
- North America: `sellingpartnerapi-na.amazon.com`
- Europe: `sellingpartnerapi-eu.amazon.com`
- Far East: `sellingpartnerapi-fe.amazon.com`

Sandbox (all regions): `sandbox.sellingpartnerapi.com`

## Common 403 Causes

| Scenario | Fix |
|----------|-----|
| Expired access token | Re-exchange the refresh token |
| Wrong token type | Grantless operation called with seller token (or vice versa) |
| Missing RDT | Restricted resource called with regular access token |
| Wrong scope | Grantless token lacks the required scope |
| Marketplace mismatch | Seller not enrolled in the target marketplace |
| Application not authorized | Seller hasn't granted your app the required roles |

## Application Roles

When registering your application, select the correct roles. Common roles:
- **Amazon Orders** — Read orders data
- **Inventory and Order Management** — Manage FBA inventory
- **Pricing** — Read and write pricing
- **Product Listing** — Create and manage listings
- **Finance and Accounting** — Access financial data
- **Buyer Communication** — Send messages to buyers
- **Direct-to-Consumer Shipping** — Buy shipping labels

Your access token only grants operations covered by the roles your application has AND the seller has authorized.

## Self-Authorization (Own Seller Account)

If building for your own seller account (not a multi-seller app), you can generate a refresh token by:
1. Registering a developer account
2. Authorizing your own application via Seller Central
3. Using the generated refresh token directly

No OAuth redirect flow needed — just use the refresh token from Seller Central directly.

## Token Refresh Strategy

```python
import time

class LWATokenManager:
    def __init__(self, client_id, client_secret, refresh_token):
        self.client_id = client_id
        self.client_secret = client_secret
        self.refresh_token = refresh_token
        self._token = None
        self._expires_at = 0

    def get_access_token(self):
        # Refresh 60 seconds early to avoid edge cases
        if time.time() >= self._expires_at - 60:
            self._refresh()
        return self._token

    def _refresh(self):
        # POST to https://api.amazon.com/auth/o2/token
        # with grant_type=refresh_token
        response = exchange_token(...)
        self._token = response["access_token"]
        self._expires_at = time.time() + response["expires_in"]
```
