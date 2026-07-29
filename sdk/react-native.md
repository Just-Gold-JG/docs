# React Native integration

Use **`@justgold/rn-sdk`** to embed the JustGold trading UI in React Native apps via **`JustGoldConnect`**.

> **New to JustGold?** Start with the unified guide: **[Partner SDK Quickstart](overview.md)** (React Native section).

## Installation

```bash
yarn add @justgold/rn-sdk react-native-webview react-native-safe-area-context
```

### Peer dependencies

| Package                          | Version |
| -------------------------------- | ------- |
| `react`                          | >= 18   |
| `react-native`                   | >= 0.70 |
| `react-native-webview`           | >= 13   |
| `react-native-safe-area-context` | >= 4    |

Wrap your app (or SDK screen) in `SafeAreaProvider`:

```tsx
import { SafeAreaProvider } from 'react-native-safe-area-context';
```

## Usage

The wrapper loads the SDK UI from JustGold CDN automatically (`GET /v1/sdk/ui-url`). **No `sdkUrl` prop required** in normal integration.

```tsx
import { SafeAreaProvider } from 'react-native-safe-area-context';
import { JustGoldConnect } from '@justgold/rn-sdk';

export function TradingScreen({ jwt, refreshToken, onDone }: Props) {
  return (
    <SafeAreaProvider>
      <JustGoldConnect
        token={jwt}
        refreshToken={refreshToken}
        sandbox={false}
        locale="en"
        theme={{ mode: 'light', primaryColor: '#2563eb' }}
        onClose={onDone}
        onSessionExpired={() => refreshSessionFromBackend()}
        onAuthRequired={() => refreshSessionFromBackend()}
        onTokensRefreshed={({ sessionToken, refreshToken: rt }) => persistTokens(sessionToken, rt)}
        onSuccess={txn => console.log('complete', txn)}
        onPaymentRequired={(payload, _resume) => {
          navigation.navigate('PartnerPayment', payload);
        }}
        onError={err => console.warn(err)}
      />
    </SafeAreaProvider>
  );
}
```

Pass `sandbox={true}` for sandbox/stage API; omit or `false` for production. The wrapper resolves API and CDN URLs internally — partners do **not** pass `apiBaseUrl`.

Token renewal is automatic (~60s before JWT expiry) when `refreshToken` is provided.

## Props

| Prop                  | Type                        | Description                                                             |
| --------------------- | --------------------------- | ----------------------------------------------------------------------- |
| `token`               | `string`                    | **Required.** Session JWT                                               |
| `refreshToken`        | `string`                    | Optional refresh token                                                  |
| `sandbox`             | `boolean`                   | `true` → sandbox API + CDN; omit/`false` → production                    |
| `sdkUrl`              | `string`                    | Optional override — signed CDN URL (default: fetch from Partner API)    |
| `sdkUiSignedUrl`      | `string`                    | Optional — from your backend token response (skips CDN fetch)             |
| `locale`              | `string`                    | `'en'` or `'ar'`                                                        |
| `theme`               | `SdkTheme`                  | Mode, colors, partner branding                                          |
| `logLevel`            | `SdkLogLevel`               | `debug` \| `info` \| `warn` \| `error` (default `warn`)                 |
| `platformFee`         | `number`                    | Flat fee for previews; omit to use `onPlatformFeeRequest`                 |
| `onClose`             | `() => void`                | User closed SDK                                                         |
| `onSessionExpired`    | `() => void`                | Token expired                                                           |
| `onAuthRequired`      | `(payload) => void`         | Re-issue session (`reason` in payload)                                  |
| `onSuccess`           | `(payload) => void`         | `TRANSACTION_COMPLETE` event                                            |
| `onTokensRefreshed`   | `(payload) => void`         | Silent renew — persist new refresh token                                |
| `onPaymentRequired`   | `(payload, resume) => void` | Open full-page payment UI — see [Full-page payment](#full-page-payment) |
| `onPlatformFeeRequest`| `(payload) => Promise<number \| null>` | Dynamic platform fee before preview                          |
| `onNavigation`        | `(payload) => void`         | In-SDK route changes (analytics)                                        |
| `onQuotePreviewed`    | `(payload) => void`         | Preview API succeeded                                                   |
| `onTransactionConfirmed` | `(payload) => void`      | Transaction created (often `Pending`)                                   |
| `onDeliveryComplete`  | `(payload) => void`         | Delivery order placed                                                   |
| `onError`             | `(payload) => void`         | `{ code, message }`                                                     |
| `onLog`               | `(payload) => void`         | Structured SDK logs                                                     |
| `onSdkEvent`          | `(event) => void`           | Catch-all for every outbound event                                      |
| `sessionRenewDelayMs` | `number`                    | **Testing only** — omit in production                                   |

## Bridge events & payloads

Every SDK → host event includes a **JSON payload example** and handler mapping:

→ **[Bridge reference](bridge-events.md)** — complete payload catalog + full React Native & Flutter callback examples

Quick example — handle payment with typed payload:

```tsx
import type { PaymentRequiredPayload } from '@justgold/rn-sdk';

onPaymentRequired={(payload: PaymentRequiredPayload, resume) => {
  // payload: { transactionId, type, amount, currency, metal, quantity }
  navigation.navigate('PartnerPayment', { payload, resume });
}}
```

## Full-page payment

When the user confirms a buy/sell, the SDK creates a **Pending** transaction and emits `PAYMENT_REQUIRED`. Your app collects payment and PATCHes transaction status via the partner HMAC API.

### Recommended flow

Keep `JustGoldConnect` **mounted**. Open payment UI on top (modal, overlay, or pushed screen). The SDK polls transaction status every 2s on its internal pending screen.

```tsx
onPaymentRequired={(payload) => navigation.navigate('PartnerPayment', payload)}

// PartnerPayment screen:
// 1. PATCH /v1/transactions/:id (HMAC)
// 2. navigation.goBack() — SDK shows success/failure automatically
```

### Unmounting during payment

If you **must unmount** the SDK during payment (e.g. native PSP SDK), remount `JustGoldConnect` with the **same** `token` / `refreshToken` after PATCH. The wrapper caches the session and sends `resumePaymentTransactionId` on the next `INIT_SESSION`. Do **not** re-fetch tokens unless expired.

### Optional: `resume(transactionId)`

The second argument to `onPaymentRequired` posts `PAYMENT_RESULT` to the SDK for instant navigation. **Not required** when the SDK stays mounted or remounts with the same session tokens.

## Metro / monorepo

If you consume the package from a monorepo path, ensure Metro can load HTML assets. See `apps/rn-b2b/metro.config.js` in this repository.

## Reference app

```bash
yarn build:sdk
yarn dev:rn-b2b
```

Full guide: [`apps/rn-b2b/README.md`](https://github.com/Just-Gold-JG/justgold.b2b)

## Next steps

→ [Bridge reference — all events with JSON payloads](bridge-events.md)  
→ [Backend authentication — session token & UI URL](session-token.md)
