# SP-API Model Catalog

All 51 API domains in `~/.sp-api-models/models/`. Use the folder name to Glob or Read the right spec.

## Sellers & Account

| Folder | Description |
|--------|-------------|
| `sellers-api-model` | Retrieve seller account info, marketplace participations |
| `authorization-api-model` | Migrate MWS auth tokens to SP-API; check delegated access |
| `application-management-api-model` | Rotate application client secrets |
| `tokens-api-model` | Create Restricted Data Tokens (RDT) for PII access |

## Orders

| Folder | Description |
|--------|-------------|
| `orders-api-model` | Core order management: getOrders, getOrder, getOrderItems, updateShipmentStatus, confirmShipment |

## Listings & Catalog

| Folder | Description |
|--------|-------------|
| `listings-items-api-model` | Create, update, delete, patch listing items; get listing issues |
| `listings-restrictions-api-model` | Check if a listing is restricted for the seller |
| `catalog-items-api-model` | Search and retrieve catalog item details (ASIN data) |
| `product-type-definitions-api-model` | Get product type schemas for listing attribute validation |
| `aplus-content-api-model` | Create and manage A+ content (enhanced brand content) |

## Inventory & FBA

| Folder | Description |
|--------|-------------|
| `fba-inventory-api-model` | Query FBA inventory summaries by FNSKU/ASIN/seller SKU |
| `fba-inbound-eligibility-api-model` | Check item eligibility for FBA inbound |
| `fba-small-and-light-api-model` | Enroll/unenroll items in FBA Small and Light program |
| `fulfillment-inbound-api-model` | Create FBA inbound shipment plans, shipments, transport |
| `amazon-warehousing-and-distribution-model` | AWD (Amazon Warehousing & Distribution) operations |
| `supply-sources-api-model` | Manage seller supply source locations |

## Fulfillment & Shipping

| Folder | Description |
|--------|-------------|
| `fulfillment-outbound-api-model` | Multi-Channel Fulfillment (MCF): create fulfillment orders, get tracking |
| `merchant-fulfillment-api-model` | Buy shipping labels for seller-fulfilled orders |
| `shipping-api-model` | Amazon Shipping: purchase labels, track shipments (v1 & v2) |
| `easy-ship-model` | Easy Ship scheduling: list pickup slots, create scheduled packages |
| `external-fulfillment` | External fulfillment integration |
| `delivery-by-amazon` | Delivery by Amazon (DBA) operations |

## Pricing & Fees

| Folder | Description |
|--------|-------------|
| `product-pricing-api-model` | Get competitive pricing, offers, buy box prices |
| `product-fees-api-model` | Estimate fees for items (referral, FBA, etc.) |
| `replenishment-api-model` | Subscribe & Save replenishment offers |

## Reports & Data

| Folder | Description |
|--------|-------------|
| `reports-api-model` | Request, check, and download seller reports (inventory, orders, finances, etc.) |
| `feeds-api-model` | Submit bulk data feeds (listings, prices, inventory, orders) via XML/JSON |
| `data-kiosk-api-model` | GraphQL-based custom analytics queries for sales, traffic, and more |
| `sales-api-model` | Get order metrics aggregated by time interval |

## Finances

| Folder | Description |
|--------|-------------|
| `finances-api-model` | List financial events, financial event groups (settlements, refunds, fees) |
| `seller-wallet-api-model` | Amazon Seller Wallet: accounts, transactions, fund transfers |
| `invoices-api-model` | VAT invoices for transactions |
| `shipment-invoicing-api-model` | Shipment invoicing for CN marketplace |

## Notifications & Messaging

| Folder | Description |
|--------|-------------|
| `notifications-api-model` | Subscribe to event notifications (order changes, listing updates, B2B offers, etc.) |
| `messaging-api-model` | Send buyer-seller messages for specific order scenarios |
| `solicitations-api-model` | Request feedback and reviews from buyers |
| `customer-feedback-api-model` | Access buyer feedback on orders |

## Vendor APIs

| Folder | Description |
|--------|-------------|
| `vendor-orders-api-model` | Vendor purchase orders (from Amazon to vendor) |
| `vendor-shipments-api-model` | Vendor shipment confirmations |
| `vendor-invoices-api-model` | Vendor invoice submission |
| `vendor-transaction-status-api-model` | Check transaction processing status |
| `vendor-direct-fulfillment-orders-api-model` | Direct fulfillment order management |
| `vendor-direct-fulfillment-shipping-api-model` | Direct fulfillment shipping labels and confirmations |
| `vendor-direct-fulfillment-inventory-api-model` | Direct fulfillment inventory updates |
| `vendor-direct-fulfillment-payments-api-model` | Direct fulfillment payment invoices |
| `vendor-direct-fulfillment-transactions-api-model` | Direct fulfillment transaction status |
| `vendor-direct-fulfillment-sandbox-test-data-api-model` | Generate sandbox test data for direct fulfillment |

## Services & Vehicles

| Folder | Description |
|--------|-------------|
| `services-api-model` | Home services: jobs, technicians, appointments |
| `vehicles-api-model` | Vehicle listing attributes and compatibility |

## Other

| Folder | Description |
|--------|-------------|
| `uploads-api-model` | Create upload destinations for images and documents |
| `application-integrations-api-model` | Notifications for application integrations |

## Finding Operations

To find a specific operation across all models:
```
Grep: pattern="\"operationId\": \"getOrders\"" path="~/.sp-api-models/models" glob="*.json"
```

To list all operations in a specific API:
```
Grep: pattern="\"operationId\"" path="~/.sp-api-models/models/orders-api-model" glob="*.json" output_mode="content"
```

## Model File Versioning

Some APIs have multiple versions (v0, v1, v2). Always check which files exist:
```
Glob: ~/.sp-api-models/models/fulfillment-inbound-api-model/*.json
```
Use the highest version available unless the user's existing code targets an older version.
