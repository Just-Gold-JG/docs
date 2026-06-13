# Buy

Generates a buy quote and places a buy order using a previously generated quote.

### Authentication

These endpoints require:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](../authentication.md) and [Request Signing](../request-signing.md).

## Buy preview

Generates a buy quote.

#### Endpoint

```http
POST /v1/buy/preview
```

#### Request body

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `metal` | string | Yes | Allowed values: `Gold`, `Silver`. |
| `amount` | string | Conditional | Numeric string. Required if `quantity` is not provided. |
| `quantity` | string | Conditional | Numeric string. Required if `amount` is not provided. |
| `customerIdentifier` | string | Yes | Partner-scoped customer identifier. If no customer exists with this identifier, one is created automatically. |

Provide exactly one of `amount` or `quantity`.
Both values must be positive when provided.

#### Sample request

```json
{
  "amount": "100",
  "metal": "Gold",
  "customerIdentifier": "cust-10293"
}
```

#### Response body

| Field | Type | Description |
| --- | --- | --- |
| `quoteId` | string | Quote identifier to pass to `POST /v1/buy`. |
| `metal` | string | `Gold` or `Silver`. |
| `currency` | string | Organization currency used for the quote. |
| `price` | string | Buy price per gram. |
| `quantity` | string | Quoted metal quantity in grams. |
| `total` | string | Quoted pre-VAT amount (quantity × price). |
| `vat` | string | VAT amount. Currently `0`. |
| `platformFee` | string | Platform fee applied on top of `total`, based on the organization's (or platform's) `platformFeeType`/`platformFee` settings. `Fixed` is a flat amount; `Percentage` is computed as a percentage of `total`. |
| `grandTotal` | string | Grand total quoted amount, i.e. `total + vat + platformFee`. |
| `expiresAt` | string | ISO 8601 timestamp (UTC) indicating when this quote expires. |

#### Sample response

```json
{
  "quoteId": "aa1c4362-7b9c-4f48-8a2b-4d4bc3e19412",
  "metal": "Gold",
  "currency": "AED",
  "price": "557.36",
  "quantity": "0.1794172488147617",
  "total": "100",
  "vat": "0",
  "platformFee": "2",
  "grandTotal": "102",
  "expiresAt": "2026-06-12T10:15:00.000Z"
}
```

#### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Quote generated successfully. |
| `400 Bad Request` | Request payload is invalid, both `amount` and `quantity` are missing, or both are provided. |
| `401 Unauthorized` | Organization context was not found for the authenticated partner. |
| `404 Not Found` | Organization was not found. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |

## Place buy order

Places a buy order using a previously generated quote.

#### Endpoint

```http
POST /v1/buy
```

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `quoteId` | string | Yes | Quote identifier returned by the buy preview endpoint. Must be a UUID. |
| `status` | string | No | Initial transaction status. One of `Pending`, `Completed`, `Failed`, `Cancelled`. Defaults to `Pending`. |
| `paymentReference` | string | No | Reference identifier for the customer's payment (e.g. payment gateway transaction ID). |
| `paymentMethod` | string | No | Payment method used by the customer (e.g. `Card`, `BankTransfer`, `Cash`). |
| `subOrgCode` | string | No | Sub-organization code to attribute the transaction to, if your organization has sub-organizations. If omitted, the quote's root organization is used. |

The customer is the one identified by `customerIdentifier` during the buy preview request — there is no need to pass a customer identifier again.

#### Sample request

```json
{
  "quoteId": "aa1c4362-7b9c-4f48-8a2b-4d4bc3e19412",
  "status": "Completed",
  "paymentReference": "pay_3f8a9c1b2d",
  "paymentMethod": "Card",
  "subOrgCode": "BR-DXB-01"
}
```

#### Response body

| Field | Type | Description |
| --- | --- | --- |
| `id` | string | Created transaction identifier. |
| `type` | string | Transaction type. Always `Buy`. |
| `metal` | string | `Gold` or `Silver`. |
| `quantity` | string | Metal quantity purchased, in grams. |
| `currency` | string | Organization currency used for the transaction. |
| `quotedPrice` | string | Price per gram from the quote at the time it was generated. |
| `status` | string | Transaction status, e.g. `Pending` or `Completed`. |
| `vat` | string | VAT amount carried over from the quote. |
| `platformFee` | string | Platform fee carried over from the quote. |
| `total` | string | Pre-VAT amount at the actual price, i.e. `quantity * actualPrice`. |
| `grandTotal` | string | Grand total charged, i.e. `total + vat + platformFee`. |

#### Sample response

```json
{
  "id": "682710cc3f1b2c7a9d5e1111",
  "type": "Buy",
  "metal": "Gold",
  "quantity": "0.1794172488147617",
  "currency": "AED",
  "quotedPrice": "557.36",
  "status": "Completed",
  "vat": "0",
  "platformFee": "2",
  "total": "100",
  "grandTotal": "102"
}
```

#### Responses

| Status | Meaning |
| --- | --- |
| `201 Created` | Buy order placed successfully. |
| `400 Bad Request` | Request payload is invalid or `quoteId` is missing. |
| `401 Unauthorized` | Organization context was not found for the authenticated partner. |
| `404 Not Found` | Organization was not found. |
| `410 Gone` | Quote has expired. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |
