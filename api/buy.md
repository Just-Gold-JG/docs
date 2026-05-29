# Buy

Places a buy order using a previously generated quote.

## Endpoint

```http
POST /v1/buy
```

## Authentication

This endpoint requires:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](../authentication.md) and [Request Signing](../request-signing.md).

## Request body

This endpoint uses the quote returned by the buy preview endpoint.

## Choosing the initial status

Partners should choose the initial `status` based on when they collect payment from the customer.

### Synchronous payment flow

Use this flow when customer payment is completed before creating the buy transaction.

```mermaid
%%{init: { "sequence": { "mirrorActors": false } }}%%
sequenceDiagram
    autonumber
    actor Customer
    participant Partner as Partner Platform
    participant JustGold as JustGold API

    Customer->>Partner: Confirm buy and complete payment
    Partner->>JustGold: POST /v1/buy with status = Completed
    JustGold-->>Partner: Transaction created as Completed
    Partner-->>Customer: Buy order completed
```

### Asynchronous payment flow

Use this flow when the partner creates the buy transaction before the customer payment result is final.

```mermaid
%%{init: { "sequence": { "mirrorActors": false } }}%%
sequenceDiagram
    autonumber
    actor Customer
    participant Partner as Partner Platform
    participant Payment as Payment Provider
    participant JustGold as JustGold API

    Customer->>Partner: Confirm buy order
    Partner->>JustGold: POST /v1/buy without status or with status = Pending
    JustGold-->>Partner: Transaction created as Pending
    Partner->>Payment: Collect payment from customer
    alt Payment succeeds
        Payment-->>Partner: Payment succeeded
        Partner->>JustGold: PATCH /v1/transactions/quote/:quote-id with status = Completed
        JustGold-->>Partner: Transaction updated to Completed
    else Payment fails
        Payment-->>Partner: Payment failed
        Partner->>JustGold: PATCH /v1/transactions/quote/:quote-id with status = Failed
        JustGold-->>Partner: Transaction updated to Failed
    end
    Partner-->>Customer: Show final payment and order status
```

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `quoteId` | string | Yes | Quote identifier returned by the preview endpoint. Must be a UUID. |
| `customerId` | string | Yes | Customer identifier. |
| `organizationCode` | string | No | Organization code to attribute the transaction to a specific organization. If omitted, the quote's root organization is used. |
| `status` | string | No | Initial transaction status. Must be `Pending` or `Completed`. If omitted, the transaction is created as `Pending`. |

## Sample request

```json
{
  "quoteId": "aa1c4362-7b9c-4f48-8a2b-4d4bc3e19412",
  "customerId": "6818744f3f1b2c7a9d5e4321",
  "status": "Completed"
}
```

## Response body

| Field | Type | Description |
| --- | --- | --- |
| `id` | string | Created transaction identifier. |

## Sample response

```json
{
  "id": "682710cc3f1b2c7a9d5e1111"
}
```

## Responses

| Status | Meaning |
| --- | --- |
| `201 Created` | Buy order placed successfully. |
| `400 Bad Request` | Request payload is invalid, `quoteId` is missing, or `customerId` is invalid. |
| `401 Unauthorized` | Organization context was not found for the authenticated partner. |
| `404 Not Found` | Customer or organization was not found. |
| `410 Gone` | Quote has expired. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |
