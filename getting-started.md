# Getting Started

Follow these steps to start your JustGold partner integration.

## 1. Get portal access

Obtain access to the JustGold developer portal from your JustGold onboarding contact.
Partner users can access the portal by creating portal credentials or by using OAuth2 / OpenID Connect SSO.

See [Portal Access](portal-access.md).

## 2. Generate API credentials

Create or retrieve:

- `client_id`
- `client_secret`

These credentials are used for all JustGold API calls.

## 3. Set up a secure backend integration

Your backend must:

- call the JustGold APIs over HTTPS
- attach `X-Client-Id` on every request
- generate `X-Timestamp` for every request
- sign every request using HMAC-SHA256

## 4. Implement the core sequence

The standard partner flow is:

1. Create customer
2. Fetch prices
3. Preview order
4. Place buy or sell order

## 5. Test before going live

Before production rollout, validate:

- request signing
- timestamp handling
- customer creation flow
- preview to order flow
- error handling and retries

Continue with [Authentication](authentication.md).
