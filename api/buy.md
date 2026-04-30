# POST /v1/buy

Places a final buy order for a customer.

## Purpose

Use this endpoint after the customer has accepted the previewed buy quote and the required payment step has been completed on your side.

## Headers

See [Authentication](../authentication.md) and [Request Signing](../request-signing.md).

## Sample request

```json
{
  "customerId": "jg_cus_01JABCDEF1234567890",
  "previewId": "jg_pre_01JABCDEF1234567890",
  "amount": 5000,
  "currency": "INR",
  "partnerReference": "buy_20260430_0001",
  "paymentReference": "pay_987654321"
}
```

## Sample response

```json
{
  "orderId": "jg_buy_01JABCDEF1234567890",
  "type": "buy",
  "status": "processing",
  "customerId": "jg_cus_01JABCDEF1234567890",
  "partnerReference": "buy_20260430_0001"
}
```

## Typical usage

- Confirm the customer and payment references before sending the request.
- Make your `partnerReference` unique to support idempotent reconciliation in your own systems.
- Store the returned `orderId` for support, audit, and settlement workflows.
