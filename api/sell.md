# Sell

Places a sell order using a previously generated quote.

## Endpoint

```http
POST /v1/sell
```

## Authentication

This endpoint requires:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](../authentication.md) and [Request Signing](../request-signing.md).

## Request body

This endpoint uses the same parameters as the sell preview request, with `quoteId` also required.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `currency` | string | Yes | Currency code. |
| `amount` | number | Conditional | Required if `quantity` is not provided. |
| `quantity` | number | Conditional | Required if `amount` is not provided. |
| `metal` | string | Yes | Metal for the order, for example `Gold` or `Silver`. |
| `quoteId` | string | Yes | Quote identifier returned by the preview endpoint. |

Either `amount` or `quantity` must be provided.

## Sample request

```json
{
  "currency": "AED",
  "amount": 100,
  "quantity": 0,
  "metal": "Gold",
  "quoteId": "6818744f3f1b2c7a9d5e4321"
}
```

## Sample response

```json
{
  "id": "6818744f3f1b2c7a9d5e4321"
}
```

## Responses

| Status | Meaning |
| --- | --- |
| `201 Created` | Sell order placed successfully. |
| `400 Bad Request` | Request payload is invalid or both `amount` and `quantity` are missing. |
| `409 Conflict` | Customer does not have enough gold or silver balance for the requested sell quantity. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |
