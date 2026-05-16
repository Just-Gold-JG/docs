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
| `metal` | string | Yes | Allowed values: `Gold`, `Silver`. |
| `amount` | string | Conditional | Numeric string. Required if `quantity` is not provided. |
| `quantity` | string | Conditional | Numeric string. Required if `amount` is not provided. |

Provide exactly one of `amount` or `quantity`.
Both values must be positive when provided.

## Sample request

```json
{
  "amount": "100",
  "metal": "Gold"
}
```

## Sample response

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

## Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Quote generated successfully. |
| `400 Bad Request` | Request payload is invalid, both `amount` and `quantity` are missing, or both are provided. |
| `401 Unauthorized` | Organization context was not found for the authenticated partner. |
| `404 Not Found` | Organization was not found. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |
