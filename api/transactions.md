# Transactions

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

See [Authentication](../authentication.md) and [Request Signing](../request-signing.md).

#### Path parameters

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `transactionId` | string | Yes | Transaction identifier returned when the transaction was created. |

#### Request body

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `status` | string | Yes | Updated transaction status. Must be one of `Pending`, `Completed`, `Failed`, or `Cancelled`. |
| `notes` | string | No | Optional notes to store with the transaction status update. |

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
  "notes": "Payment confirmed by provider reference PAY-123456"
}
```

#### Response body

| Field | Type | Description |
| --- | --- | --- |
| `id` | string | Transaction identifier. |
| `type` | string | Transaction type. One of `Buy`, `Sell`, or `Delivery`. |
| `metal` | string | `Gold` or `Silver`, when applicable. |
| `status` | string | Updated transaction status. |
| `quantity` | string | Transaction quantity. |
| `customerId` | string | Customer identifier. |
| `organizationIds` | array | Organization hierarchy associated with the transaction. |
| `currency` | string | Transaction currency. |
| `baseCurrency` | string | Base currency used for conversion. |
| `quotedPrice` | string | Quoted price per gram. |
| `actualPrice` | string | Actual price per gram. |
| `baseCurrencyQuotedPrice` | string | Quoted price in base currency. |
| `baseCurrencyActualPrice` | string | Actual price in base currency. |
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
  "baseCurrency": "USD",
  "quotedPrice": "557.36",
  "actualPrice": "557.36",
  "baseCurrencyQuotedPrice": "151.78",
  "baseCurrencyActualPrice": "151.78",
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
