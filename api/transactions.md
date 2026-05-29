# Transactions

Updates a transaction status using the quote identifier.

Transactions are usually created in `Pending` status. After processing the transaction, the partner should call this endpoint with the final status.

## Endpoint

```http
PATCH /v1/transactions/quote/:quote-id
```

## Authentication

This endpoint requires:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](../authentication.md) and [Request Signing](../request-signing.md).

## Path parameters

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `quote-id` | string | Yes | Quote identifier returned by the preview endpoint. |

## Request body

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `status` | string | Yes | Updated transaction status. Must be one of `Pending`, `Completed`, `Failed`, or `Cancelled`. |

## Valid statuses

| Status | Description |
| --- | --- |
| `Pending` | Transaction has been created but has not been completed yet. |
| `Completed` | Transaction completed successfully. |
| `Failed` | Transaction failed. |
| `Cancelled` | Transaction was cancelled. |

## Sample request

```json
{
  "status": "Completed"
}
```

## Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Transaction status updated successfully. |
| `400 Bad Request` | Request payload is invalid, `quote-id` is missing, or `status` is invalid. |
| `401 Unauthorized` | Organization context was not found for the authenticated partner. |
| `404 Not Found` | Transaction or quote was not found. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |
