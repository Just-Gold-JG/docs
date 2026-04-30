# Authentication

JustGold B2B APIs use two credentials:

- `client_id`
- `client_secret`

You can generate and manage these credentials in the JustGold developer portal.

## How it works

1. Your backend retrieves the `client_id` and `client_secret` from the developer portal.
2. Each API request includes your `client_id` in the headers.
3. Each API request is signed with HMAC using your `client_secret`.
4. JustGold verifies the signature before processing the request.

## Required headers

Send these headers on every request:

| Header | Required | Description |
| --- | --- | --- |
| `X-Client-Id` | Yes | Your JustGold client identifier. |
| `X-Timestamp` | Yes | Unix timestamp in seconds, generated at request time. |
| `X-Signature` | Yes | HMAC-SHA256 signature of the request. |
| `Content-Type` | Yes for `POST` | Use `application/json`. |

## Security requirements

- Store the `client_secret` only on your backend.
- Never expose the `client_secret` in mobile or browser applications.
- Rotate credentials immediately if you suspect they were exposed.
- Reject or retry requests if your server clock is out of sync.

Continue with [Request Signing](request-signing.md) for the exact HMAC flow.
