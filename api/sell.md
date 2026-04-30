# POST /v1/sell

Places a final sell order for a customer.

## Purpose

Use this endpoint after the customer has accepted the previewed sell quote and your platform has completed any required verification or payout checks.

## Headers

See [Authentication](../authentication.md) and [Request Signing](../request-signing.md).

## Sample request

```json
{
  "customerId": "jg_cus_01JABCDEF1234567890",
  "previewId": "jg_pre_01JABCDEF1234567890",
  "quantity": 1.25,
  "unit": "gram",
  "partnerReference": "sell_20260430_0001",
  "payoutReference": "po_123456789"
}
```

## Sample response

```json
{
  "orderId": "jg_sell_01JABCDEF1234567890",
  "type": "sell",
  "status": "processing",
  "customerId": "jg_cus_01JABCDEF1234567890",
  "partnerReference": "sell_20260430_0001"
}
```

## Typical usage

- Create and confirm a fresh preview before final sell placement.
- Validate payout readiness on your side before sending the order.
- Store the returned `orderId` and your `partnerReference` together for reconciliation.
