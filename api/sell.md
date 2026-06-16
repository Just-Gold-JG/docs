# Sell

Generates a sell quote and places a sell order using a previously generated quote.

### Authentication

These endpoints require:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](api/authentication.md) and [Request Signing](api/request-signing.md).

## Sell preview

Generates a sell quote after validating that the customer has enough sellable quantity.

#### Endpoint

```http
POST /v1/sell/preview
```

#### Request body

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `metal` | string | Yes | Allowed values: `Gold`, `Silver`. |
| `customerId` | string | Yes | Customer identifier. |
| `amount` | string | Conditional | Numeric string. Required if `quantity` is not provided. |
| `quantity` | string | Conditional | Numeric string. Required if `amount` is not provided. |

Provide exactly one of `amount` or `quantity`.
Both values must be positive when provided.

#### Sample request

```json
{
  "customerId": "6818744f3f1b2c7a9d5e4321",
  "quantity": "0.25",
  "metal": "Gold"
}
```

#### Response body

| Field | Type | Description |
| --- | --- | --- |
| `quoteId` | string | Quote identifier to pass to `POST /v1/sell`. |
| `metal` | string | `Gold` or `Silver`. |
| `currency` | string | Organization currency used for the quote. |
| `price` | string | Sell price per gram. |
| `quantity` | string | Quoted metal quantity in grams. |
| `amount` | string | Quoted pre-VAT amount. |
| `vat` | string | VAT amount. Currently `0`. |
| `total` | string | Total quoted amount. |
| `expiresAt` | string | ISO 8601 timestamp (UTC) indicating when this quote expires. |

#### Sample response

```json
{
  "quoteId": "b79a7bb2-0fd3-4dd4-a56c-7165fb1a7b55",
  "metal": "Gold",
  "currency": "AED",
  "price": "535.07",
  "quantity": "0.25",
  "amount": "133.7675",
  "vat": "0",
  "total": "133.7675",
  "expiresAt": "2026-06-12T10:15:00.000Z"
}
```

#### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Quote generated successfully. |
| `400 Bad Request` | Request payload is invalid, the customer has insufficient sellable quantity, both `amount` and `quantity` are missing, or both are provided. |
| `401 Unauthorized` | Organization context was not found for the authenticated partner. |
| `404 Not Found` | Customer or organization was not found. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |

## Place sell order

Places a sell order using a previously generated quote.

#### Endpoint

```http
POST /v1/sell
```

#### Request body

This endpoint uses the quote returned by the sell preview endpoint.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `quoteId` | string | Yes | Quote identifier returned by the preview endpoint. Must be a UUID. |
| `customerId` | string | Yes | Customer identifier. |
| `organizationCode` | string | No | Organization code to attribute the transaction to a specific organization. If omitted, the quote's root organization is used. |
| `paymentMethod` | string | No | Payment method used by the customer (e.g. `Card`, `BankTransfer`, `Cash`). |

#### Sample request

```json
{
  "quoteId": "aa1c4362-7b9c-4f48-8a2b-4d4bc3e19412",
  "customerId": "6818744f3f1b2c7a9d5e4321",
  "paymentMethod": "Card"
}
```

#### Response body

| Field | Type | Description |
| --- | --- | --- |
| `id` | string | Created transaction identifier. |

#### Sample response

```json
{
  "id": "682710cc3f1b2c7a9d5e2222"
}
```

#### Responses

| Status | Meaning |
| --- | --- |
| `201 Created` | Sell order placed successfully. |
| `400 Bad Request` | Request payload is invalid, `quoteId` is missing, or `customerId` is invalid. |
| `401 Unauthorized` | Organization context was not found for the authenticated partner. |
| `404 Not Found` | Customer or organization was not found. |
| `410 Gone` | Quote has expired. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |
