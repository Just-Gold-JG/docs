# Delivery

Creates delivery transactions for physical product delivery.

!> ⚠️ Delivery is only available for UAE partners

### Authentication

These endpoints require:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](api/authentication.md) and [Request Signing](api/request-signing.md).

## Delivery preview

Generates a delivery quote before the final transaction is created.

#### Endpoint

```http
POST /v1/delivery/preview
```

#### Request body

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `items` | array | Yes | Cart items to deliver. Must contain at least one item. |
| `items[].productId` | string | Yes | Product identifier. |
| `items[].quantity` | number | Yes | Product quantity. Must be positive. |
| `customerIdentifier` | string | Yes | Customer identifier, scoped to your organization. |
| `deliveryAddress` | object | Yes | Customer delivery address. |
| `deliveryAddress.emirate` | string | Yes | UAE emirate used to calculate delivery charges. |
| `deliveryAddress.area` | string | Yes | Area or locality. |
| `deliveryAddress.city` | string | No | City name. |
| `deliveryAddress.street` | string | Yes | Street name or number. |
| `deliveryAddress.building` | string | Yes | Building name or number. |
| `useVault` | object | No | Optional flags to use the customer's available vault balance before calculating purchase quantity. |
| `useVault.gold` | boolean | No | Use available gold vault balance for gold items. |
| `useVault.silver` | boolean | No | Use available silver vault balance for silver items. |
| `platformFee` | number | No | Flat platform fee to apply to this quote. Overrides the organisation's configured platform fee. Pass `0` to waive the fee entirely. Omit to use the default setting. |

#### Sample request

```json
{
  "customerIdentifier": "6818744f3f1b2c7a9d5e4321",
  "items": [
    {
      "productId": "682710cc3f1b2c7a9d5e3333",
      "quantity": 2
    }
  ],
  "deliveryAddress": {
    "emirate": "Dubai",
    "area": "Business Bay",
    "city": "Dubai",
    "street": "Marasi Drive",
    "building": "Bay Square"
  },
  "useVault": {
    "gold": true,
    "silver": false
  }
}
```

#### Response body

| Field | Type | Description |
| --- | --- | --- |
| `quoteId` | string | Quote identifier to pass to `POST /v1/delivery`. |
| `items` | array | Delivery quote line items. |
| `items[].productId` | string | Product identifier. |
| `items[].productName` | string | Product display name. |
| `items[].quantity` | number | Product quantity. |
| `items[].metal` | string | `Gold` or `Silver`. |
| `items[].weight` | string | Metal weight per product unit, in grams. |
| `items[].totalWeight` | string | Total metal weight for the line item, in grams. |
| `items[].mintingCharges` | string | Minting charges for the line item. |
| `items[].serviceCharges` | string | Service charges for the line item. |
| `items[].itemTotal` | string | Total charges for the line item (minting + service). |
| `metalSummary` | object | Metal totals grouped by `Gold` and/or `Silver`. |
| `metalSummary.Gold.totalQuantity` | string | Total gold weight needed for the delivery, in grams. |
| `metalSummary.Gold.vaultQuantityUsed` | string | Gold weight covered from the customer's vault, in grams. |
| `metalSummary.Gold.quantityToPurchase` | string | Gold weight still required after vault usage, in grams. |
| `metalSummary.Gold.pricePerGram` | string | Gold price per gram used for this quote. |
| `metalSummary.Gold.purchaseCost` | string | Cost of the gold weight still required to be purchased. |
| `metalSummary.Gold.totalCharges` | string | Total gold product charges (minting + service across items). |
| `metalSummary.Silver.totalQuantity` | string | Total silver weight needed for the delivery, in grams. |
| `metalSummary.Silver.vaultQuantityUsed` | string | Silver weight covered from the customer's vault, in grams. |
| `metalSummary.Silver.quantityToPurchase` | string | Silver weight still required after vault usage, in grams. |
| `metalSummary.Silver.pricePerGram` | string | Silver price per gram used for this quote. |
| `metalSummary.Silver.purchaseCost` | string | Cost of the silver weight still required to be purchased. |
| `metalSummary.Silver.totalCharges` | string | Total silver product charges (minting + service across items). |
| `deliveryCharges` | string | Delivery charge for the selected emirate. |
| `total` | string | Metal purchase cost total (sum of `metalSummary.*.purchaseCost`). |
| `vat` | string | VAT calculated on charges (minting/service charges + delivery charges). |
| `platformFee` | string | Platform fee charged to the organization, if configured. |
| `grandTotal` | string | Total amount due: `total` + `vat` + `platformFee`. |
| `currency` | string | Organization currency used for the quote. |
| `deliveryAddress` | object | Delivery address used for the quote. |
| `expiresAt` | string | ISO 8601 timestamp when `quoteId` expires and can no longer be used to create a delivery transaction. |

Only metals present in the cart are included in `metalSummary`.

#### Sample response

```json
{
  "quoteId": "aa1c4362-7b9c-4f48-8a2b-4d4bc3e19412",
  "items": [
    {
      "productId": "682710cc3f1b2c7a9d5e3333",
      "productName": "Gold Bar 1g",
      "quantity": 2,
      "metal": "Gold",
      "weight": "1",
      "totalWeight": "2",
      "mintingCharges": "15",
      "serviceCharges": "5",
      "itemTotal": "20"
    }
  ],
  "metalSummary": {
    "Gold": {
      "totalQuantity": "2",
      "vaultQuantityUsed": "1.25",
      "quantityToPurchase": "0.75",
      "pricePerGram": "250",
      "purchaseCost": "187.5",
      "totalCharges": "20"
    }
  },
  "deliveryCharges": "25",
  "total": "187.5",
  "vat": "2.25",
  "platformFee": "0",
  "grandTotal": "189.75",
  "currency": "AED",
  "deliveryAddress": {
    "emirate": "Dubai",
    "area": "Business Bay",
    "city": "Dubai",
    "street": "Marasi Drive",
    "building": "Bay Square"
  },
  "expiresAt": "2026-06-18T09:35:00.000Z"
}
```

#### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Quote generated successfully. |
| `400 Bad Request` | Request payload is invalid, the cart is empty, `customerIdentifier` is invalid, the delivery address is missing, or an item has an invalid `productId` or `quantity`. |
| `401 Unauthorized` | Organization context was not found for the authenticated partner. |
| `403 Forbidden` | Delivery transactions are not enabled for the organization. |
| `404 Not Found` | Customer, organization, or product was not found. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |

## Create delivery transaction

Creates a pending delivery transaction using a previously generated delivery quote.

#### Endpoint

```http
POST /v1/delivery
```

#### Request body

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `quoteId` | string | Yes | Quote identifier returned by `POST /v1/delivery/preview`. Must be a UUID. |
| `paymentMethod` | string | No | Payment method used by the customer (e.g. `Card`, `BankTransfer`, `Cash`). |
| `paymentReference` | string | No | Reference identifier for the payment (e.g. processor transaction ID). |
| `status` | string | No | Initial transaction status: `Pending`, `Completed`, `Failed`, or `Cancelled`. Defaults to `Pending`. |

The customer is taken from the quote generated by `POST /v1/delivery/preview` — it is not passed in this request.

#### Sample request

```json
{
  "quoteId": "aa1c4362-7b9c-4f48-8a2b-4d4bc3e19412",
  "paymentMethod": "Card"
}
```

#### Response body

| Field | Type | Description |
| --- | --- | --- |
| `id` | string | Created delivery transaction identifier. |
| `type` | string | Transaction type, always `Delivery`. |
| `status` | string | Transaction status (`Pending`, `Completed`, `Failed`, or `Cancelled`). |

#### Sample response

```json
{
  "id": "682710cc3f1b2c7a9d5e4444",
  "type": "Delivery",
  "status": "Pending"
}
```

#### Responses

| Status | Meaning |
| --- | --- |
| `201 Created` | Delivery transaction created successfully. |
| `400 Bad Request` | Request payload is invalid or `quoteId` is missing. |
| `401 Unauthorized` | Authentication failed. |
| `404 Not Found` | Customer or organization was not found. |
| `410 Gone` | Quote has expired. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |
