# Session Token

Before initializing the JustGold SDK on a mobile device, your backend must request a short-lived session token for the customer. The mobile app passes this token to the SDK at launch — your `client_secret` never leaves your backend.

### Authentication

This endpoint requires:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](api/authentication.md) and [Request Signing](api/request-signing.md).

## Request a session token

#### Endpoint

```http
POST /v1/customers/:customerIdentifier/token
```

#### Path parameters

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `customerIdentifier` | string | Yes | The partner-scoped customer identifier. If no customer exists with this identifier, one is created automatically. |

#### Request body

None.

#### Sample request

```http
POST /v1/customers/cust-10293/token
X-Client-Id: jg_partner_123
X-Timestamp: 1767225600
X-Signature: <hmac_signature>
```

#### Response body

| Field | Type | Description |
| --- | --- | --- |
| `sessionToken` | string | A signed JWT to pass to the SDK at initialization. Valid for **10 minutes**. |
| `refreshToken` | string | An opaque token used to refresh the session before it expires. |

#### Sample response

```json
{
  "sessionToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "g1ZmVyZXNoX3Rva2VuX2V4YW1wbGU..."
}
```

#### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Token issued successfully. |
| `401 Unauthorized` | HMAC signature is missing or invalid. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred. |

## Token lifetime

| Token | Lifetime | Purpose |
| --- | --- | --- |
| `sessionToken` | 10 minutes | Passed to the SDK at initialization to start a customer session. |
| `refreshToken` | — | Used by the SDK to refresh the session before the session token expires. |

Request a fresh token each time the customer opens a JustGold flow. The SDK uses the refresh token automatically to extend the session — your backend only needs to supply a new pair when the customer re-opens the flow or the refresh token itself has expired.

The refresh token is revoked as soon as the SDK is closed. A new token pair must be requested on the next launch.

## Usage

Pass the `sessionToken` to the SDK when initializing a customer session. Refer to the platform guide for the exact initialization call:

- [React Native](sdk/react-native.md)
- [Flutter](sdk/flutter.md)
