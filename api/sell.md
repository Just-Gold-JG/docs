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

This endpoint uses the quote returned by the sell preview endpoint.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `quoteId` | string | Yes | Quote identifier returned by the preview endpoint. Must be a UUID. |
| `customerId` | string | Yes | Customer identifier. |

## Sample request

```json
{
  "quoteId": "aa1c4362-7b9c-4f48-8a2b-4d4bc3e19412",
  "customerId": "6818744f3f1b2c7a9d5e4321"
}
```

## Sample response

```json
{
  "id": "682710cc3f1b2c7a9d5e2222"
}
```

## Responses

| Status | Meaning |
| --- | --- |
| `201 Created` | Sell order placed successfully. |
| `400 Bad Request` | Request payload is invalid, `quoteId` is missing, or `customerId` is invalid. |
| `401 Unauthorized` | Organization context was not found for the authenticated partner. |
| `404 Not Found` | Customer or organization was not found. |
| `410 Gone` | Quote has expired. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |
