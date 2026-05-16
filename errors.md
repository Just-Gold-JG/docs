# Errors

JustGold APIs use standard HTTP response codes.

## Common status codes

| Status | Meaning |
| --- | --- |
| `200` | Request accepted successfully. |
| `201` | Resource created successfully. |
| `400` | Request validation failed. |
| `401` | Authentication failed or signature is invalid. |
| `403` | Client is not allowed to access this resource. |
| `404` | Resource was not found. |
| `409` | The request conflicts with the current resource state. |
| `410` | Price or quote has expired. |
| `429` | Too many requests. Retry later. |
| `500` | Internal server error. |

## Sample error body

```json
{
  "code": "invalid_signature",
  "message": "The provided request signature could not be verified.",
  "requestId": "req_01JABCDEF1234567890"
}
```

## Common integration issues

| Code | Cause |
| --- | --- |
| `invalid_client` | The `X-Client-Id` is unknown or disabled. |
| `invalid_signature` | The HMAC signature does not match the received request. |
| `invalid_timestamp` | The request timestamp is missing, stale, or malformed. |
| `validation_error` | Required fields are missing or incorrectly formatted. |
| `rate_limited` | Request volume exceeded the allocated limit. |

When troubleshooting, log the request path, timestamp, raw body, and your generated signature on your backend. Never log the `client_secret`.
