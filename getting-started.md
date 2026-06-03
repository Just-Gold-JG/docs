# Getting Started

Follow these steps to start your JustGold partner integration.

## 1. Choose your integration method

JustGold supports two integration methods:

| Method | Choose this when |
| --- | --- |
| [API integration](api/overview.md) | You want full backend control and custom customer-facing screens |
| [SDK integration](sdk/overview.md) | You want to embed JustGold mobile flows faster in React Native or Flutter |

## 2. Get portal access

Obtain access to the JustGold developer portal from your JustGold onboarding contact.
Partner users can access the portal by creating portal credentials or by using OpenID Connect SSO.

See [Portal Access](portal-access.md).

## 3. Generate credentials

Create or retrieve:

- `client_id`
- `client_secret`

These credentials are used by your backend for API calls and SDK session creation.

## 4. Set up your backend

Your backend must:

- call the JustGold APIs over HTTPS
- attach `X-Client-Id` on every request
- generate `X-Timestamp` for every request
- sign every request using HMAC-SHA256
- keep partner secrets out of mobile apps

## 5. Implement the core sequence

For API integrations, the standard flow is:

1. Create customer
2. Fetch prices
3. Preview order
4. Place buy or sell order

For SDK integrations, the standard flow is:

1. Authenticate customer in your app
2. Request an SDK session from your backend
3. Launch the JustGold SDK
4. Handle completion, cancellation, and error callbacks
5. Reconcile updates with webhooks

## 6. Test before going live

Before production rollout, validate:

- request signing
- timestamp handling
- customer creation flow
- preview to order flow
- SDK launch and callback handling, if using SDK
- error handling and retries

Continue with [Authentication](authentication.md).
