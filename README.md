# JustGold B2B Partner Docs

This documentation is for partners integrating with JustGold APIs.

## What you can do

- Create and map customers
- Fetch live prices
- Preview buy and sell orders
- Place buy and sell transactions
- Secure every request with HMAC

## API summary

Available endpoints:

- `POST /v1/customers`
- `GET /v1/customers/:nationalId/holdings`
- `GET /v1/prices/latest`
- `POST /v1/buy/preview`
- `POST /v1/sell/preview`
- `POST /v1/buy`
- `POST /v1/sell`

## Start here

1. Read [Getting Started](getting-started.md)
2. Configure [Authentication](authentication.md)
3. Implement [Request Signing](request-signing.md)
4. Follow the [Integration Flow](integration-flow.md)
