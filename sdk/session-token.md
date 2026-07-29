# Backend authentication

Use **`@justgold/partner-sdk`** on your server to sign requests to the JustGold API.

## Installation

```bash
yarn add @justgold/partner-sdk
# or
npm install @justgold/partner-sdk
```

Requires **Node.js 18+**. No runtime dependencies.

## Signing requests

```ts
import { signRequest } from '@justgold/partner-sdk';

const body = JSON.stringify({
  /* ... */
});

const headers = signRequest({
  method: 'POST',
  path: '/v1/customers/acme-user-42/token',
  body,
  clientId: process.env.JUSTGOLD_CLIENT_ID!,
  secret: process.env.JUSTGOLD_CLIENT_SECRET!,
});

const res = await fetch(`${process.env.JUSTGOLD_API_BASE_URL}/v1/customers/acme-user-42/token`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    ...headers,
  },
  body,
});
```

See [`packages/partner-sdk/README.md`](https://github.com/Just-Gold-JG/justgold.b2b) for the full HMAC algorithm.

## Issuing a session token

Your mobile or web app should call **your** backend endpoint, e.g. `POST /api/justgold/session`, which:

1. Identifies the logged-in user
2. Maps them to a JustGold `customerIdentifier`
3. Calls JustGold `POST /v1/customers/{customerIdentifier}/token` with HMAC headers
4. Returns `{ sessionToken, refreshToken? }` to the client

Example response to your app:

```json
{
  "sessionToken": "eyJhbG...",
  "refreshToken": "eyJhbG..."
}
```

## Token refresh

When the SDK emits `SESSION_EXPIRED` or silently renews tokens, you may receive `TOKENS_REFRESHED` on the host with new tokens. Persist the new `refreshToken` if your integration stores it.

You can also refresh server-side using JustGold's token renew endpoint (see API docs / Postman collection).

## Example: Express route

```ts
import express from 'express';
import { signRequest } from '@justgold/partner-sdk';

const app = express();

app.post('/api/justgold/session', async (req, res) => {
  const customerId = req.user.id; // your auth
  const path = `/v1/customers/${encodeURIComponent(customerId)}/token`;
  const body = JSON.stringify({});

  const headers = signRequest({
    method: 'POST',
    path,
    body,
    clientId: process.env.JUSTGOLD_CLIENT_ID!,
    secret: process.env.JUSTGOLD_CLIENT_SECRET!,
  });

  const apiRes = await fetch(`${process.env.JUSTGOLD_API_BASE_URL}${path}`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json', ...headers },
    body,
  });

  if (!apiRes.ok) {
    return res.status(apiRes.status).json({ error: await apiRes.text() });
  }

  res.json(await apiRes.json());
});
```

## SDK UI signed URL

Mobile wrappers load the trading UI from JustGold CDN by default. The Partner API returns a **short-lived signed CloudFront URL** (TTL **1 hour**).

```http
GET /v1/sdk/ui-url?sandbox=true
Authorization: Bearer <sessionToken>
Accept: application/json
```

Query parameters:

| Param | Description |
| ----- | ----------- |
| `sandbox` | `true` for dev/stage CDN + API; omit or `false` for production |
| `version` | Optional pin, e.g. `1.0.0` (default `latest`) |

**Response:**

```json
{
  "url": "https://sdk.dev.justgold.app/latest/index.html?Expires=...&Signature=...&Key-Pair-Id=...",
  "expiresAt": "2026-07-29T15:26:03.205Z"
}
```

**Partners normally do not call this directly** — `@justgold/rn-sdk` and `justgold_sdk` fetch it automatically when `sdkUrl` is omitted. Optional server-side proxy:

```ts
app.get('/api/justgold/session', async (req, res) => {
  // ... issue sessionToken via HMAC as above ...
  const uiRes = await fetch(`${process.env.JUSTGOLD_API_BASE_URL}/v1/sdk/ui-url?sandbox=true`, {
    headers: { Authorization: `Bearer ${sessionToken}`, Accept: 'application/json' },
  });
  const { url: sdkUiSignedUrl, expiresAt: sdkUiExpiresAt } = await uiRes.json();
  res.json({ sessionToken, refreshToken, sdkUiSignedUrl, sdkUiExpiresAt });
});
```

Returns **503** when CDN signing is not configured on that environment.

CDN details (maintainers): [SDK CDN deploy](session-token.md#sdk-ui-signed-url)

## Next steps

- Web: [Web integration](../integration.md)
- React Native: [React Native integration](react-native.md)
- Flutter: [Flutter integration](flutter.md)
