# JustGold B2B Partner Docs

This documentation is for partners integrating with JustGold APIs.

## What you can do

- Access the partner portal with credentials or OAuth2 / OpenID Connect
- Create and map customers
- Fetch live prices
- Preview buy and sell orders
- Place buy and sell transactions
- Secure every request with HMAC

## API summary

Available endpoints:

- `POST /v1/customers`
- `GET /v1/customers`
- `GET /v1/customers/:nationalId/holdings`
- `GET /v1/customers/:nationalId/vault`
- `GET /v1/prices/latest`
- `POST /v1/buy/preview`
- `POST /v1/sell/preview`
- `POST /v1/buy`
- `POST /v1/sell`
- `PATCH /v1/transactions/quote/:quote-id`

## Start here

1. Read [Getting Started](getting-started.md)
2. Review [Portal Access](portal-access.md)
3. Configure [Authentication](authentication.md)
4. Implement [Request Signing](request-signing.md)
5. Follow the [Integration Flow](integration-flow.md)
