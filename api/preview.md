# Preview

Generates a quote before the final order is placed.

## Authentication

These endpoints require:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](../authentication.md) and [Request Signing](../request-signing.md).

## Buy preview

Generates a buy quote.

### Endpoint

```http
POST /v1/buy/preview
```

### Request body

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `metal` | string | Yes | Allowed values: `Gold`, `Silver`. |
| `amount` | string | Conditional | Numeric string. Required if `quantity` is not provided. |
| `quantity` | string | Conditional | Numeric string. Required if `amount` is not provided. |

Provide exactly one of `amount` or `quantity`.
Both values must be positive when provided.

### Sample request

```json
{
  "amount": "100",
  "metal": "Gold"
}
```

### Response body

| Field | Type | Description |
| --- | --- | --- |
| `quoteId` | string | Quote identifier to pass to `POST /v1/buy`. |
| `metal` | string | `Gold` or `Silver`. |
| `currency` | string | Organization currency used for the quote. |
| `price` | string | Buy price per gram. |
| `quantity` | string | Quoted metal quantity in grams. |
| `amount` | string | Quoted pre-VAT amount. |
| `vat` | string | VAT amount. Currently `0`. |
| `total` | string | Total quoted amount. |

### Sample response

```json
{
  "quoteId": "aa1c4362-7b9c-4f48-8a2b-4d4bc3e19412",
  "metal": "Gold",
  "currency": "AED",
  "price": "557.36",
  "quantity": "0.1794172488147617",
  "amount": "100",
  "vat": "0",
  "total": "100"
}
```

### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Quote generated successfully. |
| `400 Bad Request` | Request payload is invalid, both `amount` and `quantity` are missing, or both are provided. |
| `401 Unauthorized` | Organization context was not found for the authenticated partner. |
| `404 Not Found` | Organization was not found. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |

## Sell preview

Generates a sell quote after validating that the customer has enough sellable quantity.

### Endpoint

```http
POST /v1/sell/preview
```

### Request body

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `metal` | string | Yes | Allowed values: `Gold`, `Silver`. |
| `customerId` | string | Yes | Customer identifier. |
| `amount` | string | Conditional | Numeric string. Required if `quantity` is not provided. |
| `quantity` | string | Conditional | Numeric string. Required if `amount` is not provided. |

Provide exactly one of `amount` or `quantity`.
Both values must be positive when provided.

### Sample request

```json
{
  "customerId": "6818744f3f1b2c7a9d5e4321",
  "quantity": "0.25",
  "metal": "Gold"
}
```

### Response body

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

### Sample response

```json
{
  "quoteId": "b79a7bb2-0fd3-4dd4-a56c-7165fb1a7b55",
  "metal": "Gold",
  "currency": "AED",
  "price": "535.07",
  "quantity": "0.25",
  "amount": "133.7675",
  "vat": "0",
  "total": "133.7675"
}
```

### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Quote generated successfully. |
| `400 Bad Request` | Request payload is invalid, the customer has insufficient sellable quantity, both `amount` and `quantity` are missing, or both are provided. |
| `401 Unauthorized` | Organization context was not found for the authenticated partner. |
| `404 Not Found` | Customer or organization was not found. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |
