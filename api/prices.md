# Prices

Returns the latest buy and sell prices for gold and silver.

## Endpoint

```http
GET /v1/prices
```

## Authentication

This endpoint requires:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](../authentication.md) and [Request Signing](../request-signing.md).

## Sample response

```json
{
  "price24K": 557.36,
  "silverPrice": 9.3,
  "sellPrice24K": 535.07,
  "sellPriceSilver": 8.93
}
```

## Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Prices fetched successfully. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |
