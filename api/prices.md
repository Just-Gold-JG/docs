# GET /v1/prices

Returns the latest price information that your application can use to show current buy and sell rates.

## Purpose

Call this endpoint before generating a preview or displaying a live trade quote to the customer.

## Headers

See [Authentication](../authentication.md) and [Request Signing](../request-signing.md).

## Sample response

```json
{
  "currency": "INR",
  "buyPrice": 9786.45,
  "sellPrice": 9650.10,
  "unit": "gram",
  "asOf": "2026-04-30T10:15:00Z"
}
```

## Typical usage

- Refresh prices before the customer confirms a transaction.
- Cache the response only for the duration agreed during onboarding.
- Use `POST /v1/preview` to compute the actual order breakdown before final placement.
