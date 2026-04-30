# POST /v1/customers

Creates a customer in JustGold or links a partner-side customer profile for future buy and sell transactions.

## Purpose

Call this endpoint before placing an order if the customer does not already have a JustGold customer record associated with your platform.

## Headers

See [Authentication](../authentication.md) and [Request Signing](../request-signing.md).

## Sample request

This example is illustrative. Use the exact fields shared with your JustGold onboarding team if your integration contract differs.

```json
{
  "customerReference": "cust_100245",
  "fullName": "Aarav Mehta",
  "mobileNumber": "+919999999999",
  "email": "aarav@example.com"
}
```

## Sample response

```json
{
  "customerId": "jg_cus_01JABCDEF1234567890",
  "customerReference": "cust_100245",
  "status": "active"
}
```

## Typical usage

- Create a customer during first-time onboarding.
- Store the returned `customerId` in your system.
- Reuse the stored `customerId` for preview, buy, and sell flows.
