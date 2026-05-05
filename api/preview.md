# Preview

Generates a quote for a buy or sell transaction before the final order is placed.

## Endpoints

```http
POST /v1/buy/preview
POST /v1/sell/preview
```

## Authentication

This endpoint requires:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](../authentication.md) and [Request Signing](../request-signing.md).

## Request body

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `currency` | string | Yes | Currency code. |
| `amount` | number | Conditional | Required if `quantity` is not provided. |
| `quantity` | number | Conditional | Required if `amount` is not provided. |
| `metal` | string | Yes | Metal for the quote, for example `Gold` or `Silver`. |

Either `amount` or `quantity` must be provided.

## Sample request

```json
{
  "currency": "AED",
  "amount": 100,
  "quantity": 0,
  "metal": "Gold"
}
```

## Sample response

```json
{
  "quoteId": "6818744f3f1b2c7a9d5e4321"
}
```

## Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Quote generated successfully. |
| `400 Bad Request` | Request payload is invalid or both `amount` and `quantity` are missing. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |
