# Delivery

Creates delivery transactions for physical product delivery.

!> ⚠️ Delivery is only available for UAE partners

### Authentication

These endpoints require:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](../authentication.md) and [Request Signing](../request-signing.md).

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
| `customerId` | string | Yes | Customer identifier. |
| `deliveryAddress` | object | Yes | Customer delivery address. |
| `deliveryAddress.emirate` | string | Yes | UAE emirate used to calculate delivery charges. |
| `deliveryAddress.area` | string | Yes | Area or locality. |
| `deliveryAddress.street` | string | Yes | Street name or number. |
| `deliveryAddress.building` | string | Yes | Building name or number. |
| `useVault` | object | No | Optional flags to use the customer's available vault balance before calculating purchase quantity. |
| `useVault.gold` | boolean | No | Use available gold vault balance for gold items. |
| `useVault.silver` | boolean | No | Use available silver vault balance for silver items. |

#### Sample request

```json
{
  "customerId": "6818744f3f1b2c7a9d5e4321",
  "items": [
    {
      "productId": "682710cc3f1b2c7a9d5e3333",
      "quantity": 2
    }
  ],
  "deliveryAddress": {
    "emirate": "Dubai",
    "area": "Business Bay",
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
| `items[].metalWeightPerUnit` | string | Metal weight per product unit. |
| `items[].totalMetalWeight` | string | Total metal weight for the line item. |
| `items[].mintingCharges` | string | Minting charges for the line item. |
| `items[].serviceCharges` | string | Service charges for the line item. |
| `items[].itemTotal` | string | Total charges for the line item. |
| `metalSummary` | object | Metal totals grouped by `Gold` and/or `Silver`. |
| `metalSummary.Gold.totalQuantity` | string | Total gold quantity needed for the delivery. |
| `metalSummary.Gold.vaultQuantityUsed` | string | Gold quantity covered from the customer's vault. |
| `metalSummary.Gold.quantityToPurchase` | string | Gold quantity still required after vault usage. |
| `metalSummary.Gold.totalCharges` | string | Total gold product charges. |
| `metalSummary.Silver.totalQuantity` | string | Total silver quantity needed for the delivery. |
| `metalSummary.Silver.vaultQuantityUsed` | string | Silver quantity covered from the customer's vault. |
| `metalSummary.Silver.quantityToPurchase` | string | Silver quantity still required after vault usage. |
| `metalSummary.Silver.totalCharges` | string | Total silver product charges. |
| `deliveryCharges` | string | Delivery charge for the selected emirate. |
| `totalAmount` | string | Total amount including product charges and delivery charges. |
| `currency` | string | Organization currency used for the quote. |
| `deliveryAddress` | object | Delivery address used for the quote. |

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
      "metalWeightPerUnit": "1",
      "totalMetalWeight": "2",
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
      "totalCharges": "20"
    }
  },
  "deliveryCharges": "25",
  "totalAmount": "45",
  "currency": "AED",
  "deliveryAddress": {
    "emirate": "Dubai",
    "area": "Business Bay",
    "street": "Marasi Drive",
    "building": "Bay Square"
  }
}
```

#### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Quote generated successfully. |
| `400 Bad Request` | Request payload is invalid, the cart is empty, `customerId` is invalid, the delivery address is missing, or an item has an invalid `productId` or `quantity`. |
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
| `customerId` | string | Yes | Customer identifier. |

#### Sample request

```json
{
  "quoteId": "aa1c4362-7b9c-4f48-8a2b-4d4bc3e19412",
  "customerId": "6818744f3f1b2c7a9d5e4321"
}
```

#### Response body

| Field | Type | Description |
| --- | --- | --- |
| `id` | string | Created delivery transaction identifier. |

#### Sample response

```json
{
  "id": "682710cc3f1b2c7a9d5e4444"
}
```

#### Responses

| Status | Meaning |
| --- | --- |
| `201 Created` | Delivery transaction created successfully. |
| `400 Bad Request` | Request payload is invalid, `quoteId` is missing, or `customerId` is invalid. |
| `401 Unauthorized` | Authentication failed. |
| `404 Not Found` | Customer or organization was not found. |
| `410 Gone` | Quote has expired. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |
