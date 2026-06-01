# Prices

Returns the latest buy and sell prices for gold and silver.

### Endpoint

```http
GET /v1/prices/latest
```

### Authentication

This endpoint requires:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](../authentication.md) and [Request Signing](../request-signing.md).

### Sample response

```json
{
  "id": "682710cc3f1b2c7a9d5e3333",
  "price24K": 557.36,
  "sellPrice24K": 535.07,
  "silverPrice": 9.3,
  "sellPriceSilver": 8.93,
  "createdAt": "2026-05-21T08:30:00.000Z",
  "updatedAt": "2026-05-21T08:30:00.000Z"
}
```

### Response body

| Field | Type | Description |
| --- | --- | --- |
| `id` | string | Price record identifier. |
| `price24K` | number | Current buy price per gram for 24K gold. |
| `sellPrice24K` | number | Current sell price per gram for 24K gold. |
| `silverPrice` | number or null | Current buy price per gram for silver. |
| `sellPriceSilver` | number or null | Current sell price per gram for silver. |
| `createdAt` | string | ISO timestamp when this price record was created. |
| `updatedAt` | string | ISO timestamp when this price record was last updated. |

### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Prices fetched successfully. |
| `404 Not Found` | No price data is available. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |
