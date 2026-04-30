# POST /v1/preview

Previews a transaction before the final order is created.

## Purpose

Use this endpoint to validate the request and obtain an indicative transaction summary before calling `POST /v1/buy` or `POST /v1/sell`.

## Headers

See [Authentication](../authentication.md) and [Request Signing](../request-signing.md).

## Sample request

This example is illustrative. Your final request contract may include additional fields such as fees, payment metadata, or settlement details.

```json
{
  "customerId": "jg_cus_01JABCDEF1234567890",
  "type": "buy",
  "amount": 5000,
  "currency": "INR"
}
```

## Sample response

```json
{
  "previewId": "jg_pre_01JABCDEF1234567890",
  "type": "buy",
  "amount": 5000,
  "currency": "INR",
  "price": 9786.45,
  "estimatedQuantity": 0.5109,
  "expiresAt": "2026-04-30T10:20:00Z"
}
```

## Typical usage

- Generate a preview immediately before confirmation.
- Show the preview result to the customer.
- Pass any returned reference, if required by your contract, into the final buy or sell request.
