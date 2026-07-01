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
| `customerIdentifier` | string | Yes | Partner-scoped customer identifier. |
| `amount` | string | Conditional | Numeric string. Required if `quantity` is not provided. |
| `quantity` | string | Conditional | Numeric string. Required if `amount` is not provided. |
| `platformFee` | number | No | Flat platform fee to deduct on this quote. Overrides the organisation's configured platform fee. Pass `0` to waive the fee entirely. Omit to use the default setting. |

Provide exactly one of `amount` or `quantity`.
Both values must be positive when provided.

#### Sample request

```json
{
  "customerIdentifier": "cust-10293",
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
| `total` | string | Quoted pre-fee amount (quantity × price). |
| `vat` | string | VAT amount. Currently `0`. |
| `platformFee` | string | Platform fee deducted from `total`, based on the organization's (or platform's) `platformFeeType`/`platformFee` settings. `Fixed` is a flat amount; `Percentage` is computed as a percentage of `total`. |
| `grandTotal` | string | Net amount the customer receives, i.e. `total - vat - platformFee`. |
| `expiresAt` | string | ISO 8601 timestamp (UTC) indicating when this quote expires. |

#### Sample response

```json
{
  "quoteId": "b79a7bb2-0fd3-4dd4-a56c-7165fb1a7b55",
  "metal": "Gold",
  "currency": "AED",
  "price": "535.07",
  "quantity": "0.25",
  "total": "133.7675",
  "vat": "0",
  "platformFee": "2",
  "grandTotal": "131.7675",
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

Places a sell order using a previously generated quote. The customer is the one identified by `customerIdentifier` during the sell preview — there is no need to pass a customer identifier again.

#### Endpoint

```http
POST /v1/sell
```

#### Request body

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `quoteId` | string | Yes | Quote identifier returned by the sell preview endpoint. Must be a UUID. |
| `subOrgCode` | string | No | Sub-organization code to attribute the transaction to, if your organization has sub-organizations. If omitted, the quote's root organization is used. |
| `paymentReference` | string | No | Reference identifier for the customer's payment (e.g. payment gateway transaction ID). |
| `paymentMethod` | string | No | Payment method used by the customer (e.g. `Card`, `BankTransfer`, `Cash`). |

#### Sample request

```json
{
  "quoteId": "b79a7bb2-0fd3-4dd4-a56c-7165fb1a7b55",
  "paymentReference": "pay_3f8a9c1b2d",
  "paymentMethod": "Card"
}
```

#### Response body

| Field | Type | Description |
| --- | --- | --- |
| `id` | string | Created transaction identifier. |
| `type` | string | Transaction type. Always `Sell`. |
| `metal` | string | `Gold` or `Silver`. |
| `quantity` | string | Metal quantity sold, in grams. |
| `currency` | string | Organization currency used for the transaction. |
| `quotedPrice` | string | Price per gram from the quote at the time it was generated. |
| `status` | string | Transaction status, e.g. `Pending` or `Completed`. |
| `vat` | string | VAT amount carried over from the quote. |
| `platformFee` | string | Platform fee carried over from the quote. |
| `total` | string | Pre-fee amount at the actual price, i.e. `quantity × actualPrice`. |
| `grandTotal` | string | Net amount the customer receives, i.e. `total - vat - platformFee`. |

#### Sample response

```json
{
  "id": "682710cc3f1b2c7a9d5e2222",
  "type": "Sell",
  "metal": "Gold",
  "quantity": "0.25",
  "currency": "AED",
  "quotedPrice": "535.07",
  "status": "Pending",
  "vat": "0",
  "platformFee": "2",
  "total": "133.7675",
  "grandTotal": "131.7675"
}
```

#### Responses

| Status | Meaning |
| --- | --- |
| `201 Created` | Sell order placed successfully. |
| `400 Bad Request` | Request payload is invalid or `quoteId` is missing. |
| `401 Unauthorized` | Organization context was not found for the authenticated partner. |
| `404 Not Found` | Organization was not found. |
| `410 Gone` | Quote has expired. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |
