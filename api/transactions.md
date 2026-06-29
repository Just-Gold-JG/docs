# Transactions

## Get transaction

Retrieves a single transaction by its identifier.

#### Endpoint

```http
GET /v1/transactions/:transactionId
```

#### Authentication

This endpoint requires:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](api/authentication.md) and [Request Signing](api/request-signing.md).

#### Path parameters

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `transactionId` | string | Yes | Transaction identifier returned when the transaction was created. |

#### Response body

| Field | Type | Description |
| --- | --- | --- |
| `id` | string | Transaction identifier. |
| `type` | string | Transaction type. One of `Buy`, `Sell`, or `Delivery`. |
| `metal` | string | `Gold` or `Silver`, when applicable. |
| `status` | string | Transaction status. One of `Pending`, `Completed`, `Failed`, or `Cancelled`. |
| `quantity` | string | Transaction quantity in grams. |
| `customerId` | string | Customer identifier. |
| `organizationIds` | array | Organization hierarchy associated with the transaction. |
| `currency` | string | Transaction currency. |
| `quotedPrice` | string | Quoted price per gram. |
| `vat` | string | VAT amount applied to the transaction. |
| `platformFee` | string | Platform fee applied to the transaction. |
| `total` | string | Transaction total before fees. |
| `grandTotal` | string | Final total including all fees. |
| `notes` | string | Notes stored with the transaction, when set. |
| `paymentReference` | string | Reference identifier for the customer's payment, when set. |
| `paymentMethod` | string | Payment method used by the customer, when set. |
| `createdAt` | string | Transaction creation timestamp. |
| `updatedAt` | string | Transaction last update timestamp. |

#### Sample response

```json
{
  "id": "682710cc3f1b2c7a9d5e1111",
  "type": "Buy",
  "metal": "Gold",
  "status": "Completed",
  "quantity": "0.1794172488147617",
  "customerId": "6818744f3f1b2c7a9d5e4321",
  "organizationIds": [
    "6818744f3f1b2c7a9d5e9999"
  ],
  "currency": "AED",
  "quotedPrice": "557.36",
  "vat": "0.00",
  "platformFee": "0.00",
  "total": "100.00",
  "grandTotal": "100.00",
  "paymentReference": "pay_3f8a9c1b2d",
  "paymentMethod": "Card",
  "createdAt": "2026-05-30T08:15:30.000Z",
  "updatedAt": "2026-05-30T08:17:10.000Z"
}
```

#### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Transaction retrieved successfully. |
| `400 Bad Request` | `transactionId` is invalid. |
| `401 Unauthorized` | Organization context was not found for the authenticated partner. |
| `404 Not Found` | Transaction was not found. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |

---

## Update transaction status

Updates a transaction status using the transaction identifier.

Transactions are usually created in `Pending` status. After processing the transaction, the partner should call this endpoint with the final status.

#### Endpoint

```http
PATCH /v1/transactions/:transactionId
```

#### Authentication

This endpoint requires:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](api/authentication.md) and [Request Signing](api/request-signing.md).

#### Path parameters

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `transactionId` | string | Yes | Transaction identifier returned when the transaction was created. |

#### Request body

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `status` | string | Yes | Updated transaction status. Must be one of `Pending`, `Completed`, `Failed`, or `Cancelled`. |
| `notes` | string | No | Optional notes to store with the transaction status update. |
| `paymentReference` | string | No | Reference identifier for the customer's payment (e.g. payment gateway transaction ID). |
| `paymentMethod` | string | No | Payment method used by the customer (e.g. `Card`, `BankTransfer`, `Cash`). |

#### Valid statuses

| Status | Description |
| --- | --- |
| `Pending` | Transaction has been created but has not been completed yet. |
| `Completed` | Transaction completed successfully. |
| `Failed` | Transaction failed. |
| `Cancelled` | Transaction was cancelled. |

#### Sample request

```json
{
  "status": "Completed",
  "notes": "Payment confirmed by provider reference PAY-123456",
  "paymentReference": "pay_3f8a9c1b2d",
  "paymentMethod": "Card"
}
```

#### Response body

| Field | Type | Description |
| --- | --- | --- |
| `id` | string | Transaction identifier. |
| `type` | string | Transaction type. One of `Buy`, `Sell`, or `Delivery`. |
| `metal` | string | `Gold` or `Silver`, when applicable. |
| `status` | string | Updated transaction status. |
| `quantity` | string | Transaction quantity in grams. |
| `customerId` | string | Customer identifier. |
| `organizationIds` | array | Organization hierarchy associated with the transaction. |
| `currency` | string | Transaction currency. |
| `quotedPrice` | string | Quoted price per gram. |
| `vat` | string | VAT amount applied to the transaction. |
| `platformFee` | string | Platform fee applied to the transaction. |
| `total` | string | Transaction total before fees. |
| `grandTotal` | string | Final total including all fees. |
| `notes` | string | Notes stored with the transaction, when set. |
| `paymentReference` | string | Reference identifier for the customer's payment, when set. |
| `paymentMethod` | string | Payment method used by the customer, when set. |
| `createdAt` | string | Transaction creation timestamp. |
| `updatedAt` | string | Transaction last update timestamp. |

#### Sample response

```json
{
  "id": "682710cc3f1b2c7a9d5e1111",
  "type": "Buy",
  "metal": "Gold",
  "status": "Completed",
  "quantity": "0.1794172488147617",
  "customerId": "6818744f3f1b2c7a9d5e4321",
  "organizationIds": [
    "6818744f3f1b2c7a9d5e9999"
  ],
  "currency": "AED",
  "quotedPrice": "557.36",
  "vat": "0.00",
  "platformFee": "0.00",
  "total": "100.00",
  "grandTotal": "100.00",
  "paymentReference": "pay_3f8a9c1b2d",
  "paymentMethod": "Card",
  "createdAt": "2026-05-30T08:15:30.000Z",
  "updatedAt": "2026-05-30T08:17:10.000Z"
}
```

#### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Transaction status updated successfully. |
| `400 Bad Request` | Request payload is invalid, `transactionId` is invalid, or `status` is invalid. |
| `401 Unauthorized` | Organization context was not found for the authenticated partner. |
| `404 Not Found` | Transaction was not found. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |
