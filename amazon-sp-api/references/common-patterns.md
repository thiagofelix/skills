# SP-API Common Patterns

## Pagination

Most SP-API list operations return a `nextToken` (or `pageToken` in some newer APIs). Always handle it:

```python
def get_all_orders(client, created_after, marketplace_ids):
    orders = []
    next_token = None

    while True:
        params = {
            "MarketplaceIds": marketplace_ids,
            "CreatedAfter": created_after,
        }
        if next_token:
            params["NextToken"] = next_token

        response = client.get_orders(**params)
        orders.extend(response["payload"]["Orders"])

        next_token = response["payload"].get("NextToken")
        if not next_token:
            break

    return orders
```

**Note**: When using `nextToken`, you typically cannot use other filter parameters simultaneously — the token encodes the full query state. Check the specific model for restrictions.

## Rate Limiting (Throttling)

SP-API uses a token bucket algorithm. Each operation has:
- **Burst rate**: Max tokens available (requests you can fire in quick succession)
- **Restore rate**: Tokens added per second (sustained throughput)

When throttled, you receive `HTTP 429` with header `x-amzn-RateLimit-Limit` showing the rate.

**Standard retry pattern**:
```python
import time
import random

def call_with_retry(fn, max_retries=5):
    for attempt in range(max_retries):
        response = fn()
        if response.status_code == 429:
            # Exponential backoff with jitter
            wait = (2 ** attempt) + random.uniform(0, 1)
            time.sleep(wait)
            continue
        return response
    raise Exception("Max retries exceeded")
```

**Common rate limits** (check the model's `x-amzn-api-sandbox` or SP-API docs for exact values):
- `getOrders`: 0.0167 req/s restore rate (1 per minute), burst 20
- `getOrder`: 0.5 req/s, burst 30
- `getOrderItems`: 0.5 req/s, burst 30
- `createReport`: 0.0167 req/s, burst 15
- `getReport`: 2 req/s, burst 15

Always check the current limits in the model file — they change over time.

## Error Handling

SP-API errors follow a standard shape:

```json
{
  "errors": [
    {
      "code": "InvalidInput",
      "message": "...",
      "details": "..."
    }
  ]
}
```

Common error codes:

| Code | Meaning |
|------|---------|
| `InvalidInput` | Bad request parameters |
| `NotFound` | Resource doesn't exist |
| `Unauthorized` | Invalid or expired access token |
| `AccessDenied` | Missing role/permission or wrong token type |
| `QuotaExceeded` | Rate limit hit (HTTP 429) |
| `ServiceUnavailable` | Transient SP-API issue — retry with backoff |
| `InternalFailure` | SP-API server error — retry |
| `InvalidOrderState` | Order state doesn't allow the operation |

## Sandbox Testing

Use `sandbox.sellingpartnerapi.com` as the base URL for all endpoints. The path and request format are identical to production.

**Sandbox responses are predefined** — you cannot create real data. Look at the model file's `x-amazon-spds-sandbox-behaviors` extension for available test cases:

```json
"x-amazon-spds-sandbox-behaviors": [
  {
    "request": {
      "parameters": {
        "MarketplaceIds": { "value": ["ATVPDKIKX0DER"] }
      }
    },
    "response": {
      "status": 200,
      "body": { ... }
    }
  }
]
```

You must send the exact request parameters specified in the sandbox behavior to get that canned response.

## Marketplace IDs

| Marketplace | ID |
|-------------|-----|
| US (NA) | `ATVPDKIKX0DER` |
| Canada (NA) | `A2EUQ1WTGCTBG2` |
| Mexico (NA) | `A1AM78C64UM0Y8` |
| Brazil (NA) | `A2Q3Y263D00KWC` |
| UK (EU) | `A1F83G8C2ARO7P` |
| Germany (EU) | `A1PA6795UKMFR9` |
| France (EU) | `A13V1IB3VIYZZH` |
| Italy (EU) | `APJ6JRA9NG5V4` |
| Spain (EU) | `A1RKKUPIHCS9HS` |
| Netherlands (EU) | `A1805IZSGTT6HS` |
| Sweden (EU) | `A2NODRKZP88ZB9` |
| Poland (EU) | `A1C3SOZRARQ6R3` |
| Belgium (EU) | `AMEN7PMS3EDWL` |
| Turkey (EU) | `A33AVAJ2PDY3EV` |
| UAE (EU) | `A2VIGQ35RCS4UG` |
| Saudi Arabia (EU) | `A17E79C6D8DWNP` |
| Egypt (EU) | `ARBP9OOSHTCHU` |
| India (EU) | `A21TJRUUN4KGV` |
| Japan (FE) | `A1VC38T7YXB528` |
| Australia (FE) | `A39IBJ37TRP1C6` |
| Singapore (FE) | `A19VAU5U5O7RUS` |

## Reports Pattern

Reports are async — request, poll, then download:

```python
# 1. Create report request
report_response = client.create_report(
    reportType="GET_FLAT_FILE_OPEN_LISTINGS_DATA",
    marketplaceIds=["ATVPDKIKX0DER"]
)
report_id = report_response["reportId"]

# 2. Poll until DONE (or FATAL)
while True:
    status = client.get_report(reportId=report_id)
    if status["processingStatus"] in ("DONE", "FATAL", "CANCELLED"):
        break
    time.sleep(30)

# 3. Download the document
if status["processingStatus"] == "DONE":
    doc = client.get_report_document(
        reportDocumentId=status["reportDocumentId"]
    )
    # doc["url"] is a presigned S3 URL, valid for ~5 minutes
    # doc["compressionAlgorithm"] may be "GZIP"
    content = requests.get(doc["url"]).content
```

## Feeds Pattern

Feeds are also async — create, upload, submit, poll:

```python
# 1. Create upload destination
dest = client.create_feed_document(contentType="text/xml; charset=UTF-8")
# dest["url"] is a presigned S3 PUT URL
# dest["feedDocumentId"] is the ID to reference later

# 2. Upload the feed content
requests.put(dest["url"], data=feed_content, headers={"Content-Type": "text/xml; charset=UTF-8"})

# 3. Submit the feed
feed = client.create_feed(
    feedType="POST_PRODUCT_DATA",
    marketplaceIds=["ATVPDKIKX0DER"],
    inputFeedDocumentId=dest["feedDocumentId"]
)

# 4. Poll until DONE
while True:
    status = client.get_feed(feedId=feed["feedId"])
    if status["processingStatus"] in ("DONE", "FATAL", "CANCELLED"):
        break
    time.sleep(60)

# 5. Download the feed processing report
result_doc = client.get_feed_document(
    feedDocumentId=status["resultFeedDocumentId"]
)
```

## Notifications Pattern

Subscribing to event notifications (e.g., order status changes):

```python
# 1. Create a destination (where to send events)
# For SQS: get an EventBridge destination via grantless token
destination = client.create_destination(
    name="my-sqs-destination",
    resourceSpecification={
        "sqs": { "arn": "arn:aws:sqs:us-east-1:..." }
    }
)

# 2. Create a subscription
subscription = client.create_subscription(
    notificationType="ORDER_STATUS_CHANGE",
    # or: ANY_OFFER_CHANGED, LISTINGS_ITEM_STATUS_CHANGE,
    #     ITEM_PRODUCT_TYPE_CHANGE, etc.
    payload={
        "subscriptionId": "my-subscription",
        "destinationId": destination["destinationId"]
    }
)
```

Notification events arrive as JSON in your SQS queue or EventBridge bus.

## Listings Patch vs Put

- **PUT** (`putListingsItem`): Full replacement. Provide all attributes for the product type. Missing attributes may be cleared.
- **PATCH** (`patchListingsItem`): Partial update. Send only the attributes to add/replace/delete using JSON Patch operations.

For patches, use the `productType` from `getListingsItem` to ensure your patch is compatible with the current listing's type schema.
