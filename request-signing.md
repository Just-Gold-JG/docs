# Request Signing

Every request to JustGold must be signed with HMAC-SHA256 using your `client_secret`.

## Signature inputs

Build the string to sign with these exact values:

```text
{HTTP_METHOD}
{REQUEST_PATH}
{TIMESTAMP}
{RAW_REQUEST_BODY}
```

Rules:

- `HTTP_METHOD` must be uppercase, for example `GET` or `POST`.
- `REQUEST_PATH` must contain only the path, for example `/v1/prices`.
- `TIMESTAMP` must match the value sent in `X-Timestamp`.
- `RAW_REQUEST_BODY` must be the exact JSON string sent over the wire.
- For requests without a body, use an empty string for `RAW_REQUEST_BODY`.

## Signature algorithm

1. Build the string to sign.
2. Compute `HMAC-SHA256(string_to_sign, client_secret)`.
3. Encode the digest as lowercase hexadecimal.
4. Send the result in `X-Signature`.

## Example

### Request

```http
POST /v1/preview
X-Client-Id: jg_partner_123
X-Timestamp: 1767225600
Content-Type: application/json
```

```json
{"type":"buy","amount":5000,"currency":"INR"}
```

### String to sign

```text
POST
/v1/preview
1767225600
{"type":"buy","amount":5000,"currency":"INR"}
```

### Node.js example

```js
import crypto from "node:crypto";

function createSignature({ method, path, timestamp, body, clientSecret }) {
  const rawBody = body ?? "";
  const stringToSign = [
    method.toUpperCase(),
    path,
    String(timestamp),
    rawBody,
  ].join("\n");

  return crypto
    .createHmac("sha256", clientSecret)
    .update(stringToSign, "utf8")
    .digest("hex");
}

const timestamp = Math.floor(Date.now() / 1000);
const body = JSON.stringify({
  type: "buy",
  amount: 5000,
  currency: "INR",
});

const signature = createSignature({
  method: "POST",
  path: "/v1/preview",
  timestamp,
  body,
  clientSecret: process.env.JUSTGOLD_CLIENT_SECRET,
});

console.log({
  "X-Client-Id": process.env.JUSTGOLD_CLIENT_ID,
  "X-Timestamp": timestamp,
  "X-Signature": signature,
});
```

## Verification checklist

- Sign the exact JSON payload you send.
- Do not pretty-print a body after generating the signature.
- Generate a fresh timestamp for every request.
- Keep your server time synchronized with NTP.
- Treat signatures as invalid if either the path or body changes.
