# Terms & Conditions

Fetch the terms and conditions document JustGold publishes for your organization, in English or Arabic.

## Download the document

#### Endpoint

```http
GET /v1/organizations/terms
```

#### Authentication

Either:
- `Authorization: Bearer <sessionToken>` — SDK session JWT
- HMAC headers (`X-Client-Id`, `X-Timestamp`, `X-Signature`) — partner backend

See [Authentication](api/authentication.md) and [Request Signing](api/request-signing.md).

#### Query parameters

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `locale` | string | No | `en` or `ar`. Defaults to `en`. Falls back to `en` if the requested language has not been published. |

#### Sample response

```json
{
  "url": "https://justgold-content.s3.ap-southeast-1.amazonaws.com/org-terms/...&X-Amz-Signature=...",
  "expiresIn": 300,
  "locale": "ar",
  "requestedLocale": "ar",
  "fileName": "justgold-terms-ar.pdf",
  "contentType": "application/pdf",
  "sizeBytes": 52049,
  "uploadedAt": "2026-08-25T15:32:45.721Z"
}
```

#### Response body

| Field | Type | Description |
| --- | --- | --- |
| `url` | string | Pre-signed download link. Expires after `expiresIn` seconds. |
| `expiresIn` | number | Lifetime of `url` in seconds. |
| `locale` | string | Language served. |
| `requestedLocale` | string | Language requested. Differs from `locale` when a fallback was served. |
| `fileName` | string | File name of the document. |
| `contentType` | string | `application/pdf`, `application/msword`, or the DOCX equivalent. |
| `sizeBytes` | number | Size of the document in bytes. |
| `uploadedAt` | string | ISO timestamp of when the document was last published. |

#### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Document found. |
| `401 Unauthorized` | Missing or invalid credentials. |
| `404 Not Found` | No document has been published for your organization. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |
