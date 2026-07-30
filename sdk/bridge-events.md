# Bridge events & payloads

All platforms use the same JSON message envelope. Platform wrappers (`justgold_sdk`, `@justgold/rn-sdk`) translate bridge messages into typed callbacks — partners normally implement **callbacks**, not raw `postMessage`.

```json
{ "type": "EVENT_NAME", "payload": {} }
```

Some host → SDK messages omit `payload` (e.g. `PAYMENT_RESULT` uses top-level `transactionId`).

---

## Quick reference

### Host → SDK (you send — wrappers send most of these automatically)

| Event                   | Partner action?                                  | Flutter / wrapper                              |
| ----------------------- | ------------------------------------------------ | ---------------------------------------------- |
| `INIT_SESSION`          | Pass props — wrapper sends                       | Automatic on load                              |
| `PAYMENT_RESULT`        | Optional fast-path after payment                 | `resume(transactionId)` in `onPaymentRequired` |
| `UPDATE_PLATFORM_FEE`   | Optional runtime fee change                      | `setPlatformFee` (ref API where exposed)       |
| `PLATFORM_FEE_RESPONSE` | Only if custom host (not using wrapper callback) | Wrapper handles via `onPlatformFeeRequest`     |
| `SET_LOG_LEVEL`         | Optional                                         | Pass `logLevel` prop or runtime message        |

### SDK → Host (you receive — implement callbacks)

| Event                   | Flutter callback                           | Required?                               |
| ----------------------- | ------------------------------------------ | --------------------------------------- |
| `WEBVIEW_READY`         | — (wrapper handles)                        | —                                       |
| `SESSION_STARTED`       | `onSdkEvent`                               | Optional analytics                      |
| `AUTH_REQUIRED`         | `onAuthRequired`                           | **Yes** (re-issue session)              |
| `SESSION_EXPIRED`       | `onSessionExpired`                         | **Yes**                                 |
| `TOKENS_REFRESHED`      | `onTokensRefreshed`                        | **Recommended** (persist refresh token) |
| `LOG`                   | `onLog`                                    | Optional                                |
| `PLATFORM_FEE_REQUEST`  | `onPlatformFeeRequest`                     | If dynamic fee                          |
| `QUOTE_PREVIEWED`       | `onSdkEvent` / `onQuotePreviewed` (RN/Web) | Optional                                |
| `TRANSACTION_CONFIRMED` | `onSdkEvent` / `onTransactionConfirmed`    | Optional                                |
| `NAVIGATION`            | `onSdkEvent` / `onNavigation`              | Optional analytics                      |
| `PAYMENT_REQUIRED`      | `onPaymentRequired`                        | **Yes** (payment flow)                  |
| `PAYMENT_PENDING_CLEAR` | — (wrapper internal)                       | —                                       |
| `PAYMENT_DISMISSED`     | — (wrapper internal)                       | —                                       |
| `TRANSACTION_COMPLETE`  | `onSuccess`                                | Optional                                |
| `DELIVERY_COMPLETE`     | `onSdkEvent` / `onDeliveryComplete`        | Optional                                |
| `CLOSE`                 | `onClose`                                  | **Yes**                                 |
| `ERROR`                 | `onError`                                  | Recommended                             |

> **Catch-all:** `onSdkEvent` (Flutter) receives **every** outbound event as a `Map` if you prefer one handler.

---

## Flutter integration example (all callbacks)

Complete pattern for partner apps using `justgold_sdk`:

```dart
import 'package:flutter/material.dart';
import 'package:justgold_sdk/justgold_sdk.dart';

class TradingScreen extends StatefulWidget {
  const TradingScreen({
    super.key,
    required this.initialToken,
    required this.initialRefreshToken,
  });

  final String initialToken;
  final String initialRefreshToken;

  @override
  State<TradingScreen> createState() => _TradingScreenState();
}

class _TradingScreenState extends State<TradingScreen> {
  late String _token = widget.initialToken;
  String? _refreshToken = widget.initialRefreshToken;

  Future<void> _reissueSession() async {
    // Call partner backend → POST /v1/customers/{id}/token (HMAC on server)
    final next = await partnerBackend.fetchJustGoldSession();
    if (!mounted) return;
    setState(() {
      _token = next.sessionToken;
      _refreshToken = next.refreshToken;
    });
    // await secureStorage.write(key: 'jg_refresh', value: next.refreshToken);
  }

  @override
  Widget build(BuildContext context) {
    return JustGoldConnect(
      token: _token,
      refreshToken: _refreshToken,
      sandbox: false, // true for sandbox API
      locale: 'en',
      theme: const SdkTheme(mode: SdkThemeMode.light, primaryColor: '#2563eb'),
      logLevel: 'warn',

      // --- Required / recommended callbacks ---

      onClose: () => Navigator.of(context).pop(),

      onSessionExpired: _reissueSession,
      onAuthRequired: (_) => _reissueSession(),

      onTokensRefreshed: (payload) {
        final sessionToken = payload['sessionToken'] as String?;
        final refreshToken = payload['refreshToken'] as String?;
        if (sessionToken == null) return;
        setState(() {
          _token = sessionToken;
          _refreshToken = refreshToken;
        });
        // Persist refreshToken in flutter_secure_storage
      },

      onPaymentRequired: (payload, resume) {
        Navigator.of(context).push(
          MaterialPageRoute(
            builder: (_) => PartnerPaymentPage(
              payload: payload,
              onDone: () {
                Navigator.of(context).pop();
                // Optional fast-path — not required if SDK stays mounted (polls API):
                resume(payload.transactionId);
              },
            ),
          ),
        );
      },

      onSuccess: (payload) {
        debugPrint('Transaction complete: $payload');
      },

      onError: (err) {
        debugPrint('SDK error [${err['code']}]: ${err['message']}');
      },

      onLog: (log) {
        debugPrint('[JustGold ${log['level']}] ${log['message']}');
      },

      onPlatformFeeRequest: (payload) async {
        // Return flat fee in org currency, or null for org default
        return partnerBackend.fetchPlatformFee(
          operation: payload['operation'] as String,
          metal: payload['metal'] as String?,
        );
      },

      // --- Optional: raw event stream for analytics ---

      onSdkEvent: (event) {
        switch (event['type']) {
          case 'NAVIGATION':
            debugPrint('SDK route: ${event['payload']}');
          case 'QUOTE_PREVIEWED':
          case 'TRANSACTION_CONFIRMED':
          case 'DELIVERY_COMPLETE':
            debugPrint('SDK event: $event');
        }
      },
    );
  }
}
```

**Payment return (recommended — SDK stays mounted):**

1. SDK shows internal pending screen and polls `GET /transactions/:id` every 2s
2. Partner opens payment UI on top
3. Partner backend `PATCH /v1/transactions/:id` (HMAC) → `Completed` or `Failed`
4. Partner closes payment UI (`Navigator.pop`)
5. SDK detects terminal status automatically — **no new JWT** unless session expired

---

## Host → SDK messages

### `INIT_SESSION`

Sent by the platform wrapper when the UI is ready (`WEBVIEW_READY`) and when session props change. Partners pass credentials via widget props — you do not construct this manually in normal integration.

| Field                        | Type             | Required | Description                                             |
| ---------------------------- | ---------------- | -------- | ------------------------------------------------------- |
| `type`                       | `"INIT_SESSION"` | Yes      |                                                         |
| `token`                      | `string`         | Yes      | Session JWT from your backend                           |
| `refreshToken`               | `string`         | No       | Enables silent renew (~60s before JWT `exp`)            |
| `apiBaseUrl`                 | `string`         | Yes      | Set by wrapper from `sandbox` flag                      |
| `locale`                     | `string`         | No       | `en` or `ar`                                            |
| `theme`                      | `object`         | No       | See [Theming](#theming-theme-object)                    |
| `safeAreaInsets`             | `object`         | No       | `{ top, bottom, left, right }` — native only            |
| `platformFee`                | `number`         | No       | Flat platform fee for preview APIs                      |
| `useHostPlatformFee`         | `boolean`        | No       | `true` when `onPlatformFeeRequest` is set               |
| `logLevel`                   | `string`         | No       | `debug` \| `info` \| `warn` \| `error`                  |
| `sessionRenewDelayMs`        | `number`         | No       | **Testing only** — fixed renew delay                    |
| `resumePaymentTransactionId` | `string`         | No       | **Wrapper internal** — after SDK remount during payment |

**Example:**

```json
{
  "type": "INIT_SESSION",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "k7xR2mN9pQ4vL8wY1zA3bC5dE6fG0hI",
  "apiBaseUrl": "https://api.partner.justgold.app",
  "locale": "en",
  "sandbox": false,
  "theme": {
    "mode": "light",
    "primaryColor": "#2563eb",
    "branding": { "partnerName": "Your Bank", "logoUrl": "https://cdn.example.com/logo.png" }
  },
  "safeAreaInsets": { "top": 47, "bottom": 34, "left": 0, "right": 0 },
  "platformFee": 5.0,
  "logLevel": "warn"
}
```

**After remount during payment (wrapper adds):**

```json
{
  "type": "INIT_SESSION",
  "token": "eyJ...",
  "refreshToken": "k7x...",
  "apiBaseUrl": "https://api.partner.justgold.app",
  "resumePaymentTransactionId": "674a1b2c3d4e5f6789012345"
}
```

---

### `PAYMENT_RESULT`

Optional fast-path from host → SDK after the partner completes payment and PATCHes transaction status. **Not required** when the SDK stays mounted — polling handles the result.

```json
{
  "type": "PAYMENT_RESULT",
  "transactionId": "674a1b2c3d4e5f6789012345"
}
```

**Flutter:**

```dart
onPaymentRequired: (payload, resume) async {
  await openPartnerPayment(payload);
  resume(payload.transactionId); // optional — triggers immediate status check
},
```

---

### `UPDATE_PLATFORM_FEE`

Change platform fee without re-sending the full session.

```json
{ "type": "UPDATE_PLATFORM_FEE", "platformFee": 5.0 }
```

Clear override (use org default / dynamic fetch):

```json
{ "type": "UPDATE_PLATFORM_FEE", "platformFee": null }
```

---

### `PLATFORM_FEE_RESPONSE`

Reply to SDK `PLATFORM_FEE_REQUEST`. **Platform wrappers send this automatically** when you implement `onPlatformFeeRequest` — you normally do not send this yourself.

```json
{
  "type": "PLATFORM_FEE_RESPONSE",
  "requestId": "550e8400-e29b-41d4-a716-446655440000",
  "platformFee": 5.0
}
```

Use `null` to omit override (API org default):

```json
{
  "type": "PLATFORM_FEE_RESPONSE",
  "requestId": "550e8400-e29b-41d4-a716-446655440000",
  "platformFee": null
}
```

---

### `SET_LOG_LEVEL`

```json
{ "type": "SET_LOG_LEVEL", "level": "debug" }
```

---

## SDK → Host events

### `WEBVIEW_READY`

UI bundle mounted. **Wrapper handles** — sends `INIT_SESSION` automatically (with retries on iOS).

```json
{ "type": "WEBVIEW_READY" }
```

---

### `SESSION_STARTED`

Session applied after successful `INIT_SESSION`.

```json
{
  "type": "SESSION_STARTED",
  "payload": {
    "locale": "en",
    "sandbox": false
  }
}
```

---

### `AUTH_REQUIRED`

Authentication failed — re-issue session from partner backend.

```json
{
  "type": "AUTH_REQUIRED",
  "payload": {
    "reason": "unauthorized"
  }
}
```

| `reason`             | When                      |
| -------------------- | ------------------------- |
| `session_expired`    | JWT expired (legacy path) |
| `token_renew_failed` | Silent renew failed       |
| `unauthorized`       | API returned 401          |

Also followed by `SESSION_EXPIRED`. Implement `onAuthRequired` and/or `onSessionExpired`.

---

### `SESSION_EXPIRED`

JWT expired and could not be renewed. Fetch a new token pair from partner backend and update widget props.

```json
{ "type": "SESSION_EXPIRED" }
```

---

### `TOKENS_REFRESHED`

Silent renewal succeeded. **Persist the new refresh token** — it rotates on every renew.

```json
{
  "type": "TOKENS_REFRESHED",
  "payload": {
    "sessionToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "nEwR3fR3shT0k3nAfT3rR0t4t3..."
  }
}
```

**Flutter:**

```dart
onTokensRefreshed: (payload) {
  secureStorage.write(key: 'jg_refresh', value: payload['refreshToken']);
  setState(() {
    _token = payload['sessionToken'] as String;
    _refreshToken = payload['refreshToken'] as String?;
  });
},
```

---

### `LOG`

Structured SDK log line when `logLevel` allows.

```json
{
  "type": "LOG",
  "payload": {
    "level": "info",
    "message": "Platform fee for preview",
    "timestamp": "2026-07-17T10:30:00.000Z",
    "context": {
      "operation": "buy",
      "platformFee": 5
    }
  }
}
```

---

### `PLATFORM_FEE_REQUEST`

SDK needs platform fee before calling preview API. Respond via `onPlatformFeeRequest` (wrapper sends `PLATFORM_FEE_RESPONSE`).

```json
{
  "type": "PLATFORM_FEE_REQUEST",
  "payload": {
    "requestId": "550e8400-e29b-41d4-a716-446655440000",
    "operation": "buy",
    "metal": "Gold",
    "amount": "500"
  }
}
```

| Field       | Values                                   |
| ----------- | ---------------------------------------- |
| `operation` | `buy` \| `sell` \| `delivery`            |
| `metal`     | `Gold` \| `Silver` (buy/sell)            |
| `amount`    | String amount in org currency (buy/sell) |
| `quantity`  | String grams (optional)                  |

---

### `QUOTE_PREVIEWED`

Preview API succeeded.

**Buy example:**

```json
{
  "type": "QUOTE_PREVIEWED",
  "payload": {
    "operation": "buy",
    "quoteId": "q_buy_abc123",
    "metal": "Gold",
    "amount": 512.5,
    "platformFee": 5.0,
    "currency": "AED"
  }
}
```

**Delivery example:**

```json
{
  "type": "QUOTE_PREVIEWED",
  "payload": {
    "operation": "delivery",
    "quoteId": "q_del_xyz789",
    "amount": 1250.0,
    "platformFee": 7.5,
    "currency": "AED"
  }
}
```

---

### `TRANSACTION_CONFIRMED`

Buy/sell/delivery confirm created a transaction (usually `Pending` for native payment handoff).

**Buy (pending payment):**

```json
{
  "type": "TRANSACTION_CONFIRMED",
  "payload": {
    "transactionId": "674a1b2c3d4e5f6789012345",
    "type": "buy",
    "status": "Pending",
    "amount": 512.5,
    "currency": "AED"
  }
}
```

**Sell:**

```json
{
  "type": "TRANSACTION_CONFIRMED",
  "payload": {
    "transactionId": "674a1b2c3d4e5f6789012346",
    "type": "sell",
    "status": "Pending",
    "amount": 480.0,
    "currency": "AED"
  }
}
```

**Delivery:**

```json
{
  "type": "TRANSACTION_CONFIRMED",
  "payload": {
    "transactionId": "674a1b2c3d4e5f6789012347",
    "type": "delivery",
    "status": "Pending",
    "amount": 1250.0,
    "currency": "AED"
  }
}
```

---

### `NAVIGATION`

In-SDK route changed — useful for analytics.

```json
{
  "type": "NAVIGATION",
  "payload": {
    "route": "/buy/confirm",
    "previousRoute": "/buy"
  }
}
```

---

### `PAYMENT_REQUIRED`

SDK created a **Pending** transaction. The partner must collect payment and PATCH status via **partner backend HMAC API**.

```json
{
  "type": "PAYMENT_REQUIRED",
  "payload": {
    "transactionId": "674a1b2c3d4e5f6789012345",
    "type": "buy",
    "amount": 512.5,
    "currency": "AED",
    "metal": "Gold",
    "quantity": 2.5
  }
}
```

| `payload.type` | Partner action                                 |
| -------------- | ---------------------------------------------- |
| `buy`          | Collect payment → PATCH `Completed` / `Failed` |
| `sell`         | Confirm payout / bank credit → PATCH status    |
| `delivery`     | Collect delivery payment → PATCH status        |

**Partner backend (HMAC — not from mobile SDK):**

```http
PATCH /v1/transactions/674a1b2c3d4e5f6789012345
Content-Type: application/json
Authorization: Bearer <HMAC headers>

{
  "status": "Completed",
  "paymentReference": "partner-psp-ref-12345",
  "paymentMethod": "bank_transfer"
}
```

---

### `PAYMENT_PENDING_CLEAR`

SDK detected terminal payment status (`Completed` / `Failed`) while polling. **Wrapper internal** — clears remount recovery state. Partners do not handle this.

```json
{ "type": "PAYMENT_PENDING_CLEAR" }
```

---

### `PAYMENT_DISMISSED`

User left the payment result screen. **Wrapper internal.**

```json
{
  "type": "PAYMENT_DISMISSED",
  "transactionId": "674a1b2c3d4e5f6789012345"
}
```

---

### `TRANSACTION_COMPLETE`

Buy or sell reached terminal success (result screen).

```json
{
  "type": "TRANSACTION_COMPLETE",
  "payload": {
    "txnId": "674a1b2c3d4e5f6789012345",
    "type": "buy",
    "amount": 512.5
  }
}
```

Maps to Flutter `onSuccess`.

---

### `DELIVERY_COMPLETE`

Delivery order placed.

```json
{
  "type": "DELIVERY_COMPLETE",
  "payload": {
    "orderId": "674a1b2c3d4e5f6789012347",
    "transactionId": "674a1b2c3d4e5f6789012347"
  }
}
```

---

### `CLOSE`

User tapped close in the SDK UI.

```json
{ "type": "CLOSE" }
```

---

### `ERROR`

Unrecoverable or reported SDK/API error (non-auth).

```json
{
  "type": "ERROR",
  "payload": {
    "code": "NETWORK",
    "message": "Unable to reach API"
  }
}
```

Common `code` values: API error codes from the Partner API, `NETWORK`, validation errors. Auth failures use `AUTH_REQUIRED` instead.

---

## Full-page partner payment (recommended)

1. SDK emits `PAYMENT_REQUIRED` and navigates to an internal pending screen (polls every 2s).
2. Partner opens a **full-screen payment route on top**, keeping `JustGoldConnect` mounted.
3. Partner backend PATCHes transaction status via HMAC.
4. Partner closes the payment screen — SDK shows success/failure automatically.

If the partner **must unmount** the SDK during payment, remount with the **same** `token` and `refreshToken`. The wrapper adds `resumePaymentTransactionId` to the next `INIT_SESSION`.

---

## Native-only: `API_FETCH` / `API_RESPONSE`

Used internally by `@justgold/rn-sdk` and `justgold_sdk` to proxy HTTP from the WebView through native TLS stacks. **Web hosts and partners do not implement this.**

**UI → native:**

```json
{
  "type": "API_FETCH",
  "requestId": "550e8400-e29b-41d4-a716-446655440000",
  "url": "https://api.partner.justgold.app/v1/buy/preview",
  "method": "POST",
  "headers": {
    "Authorization": "Bearer eyJ...",
    "Content-Type": "application/json",
    "Accept": "application/json"
  },
  "body": "{\"metal\":\"Gold\",\"customerIdentifier\":\"cust_123\",\"amount\":\"500\"}"
}
```

**Native → UI:**

```json
{
  "type": "API_RESPONSE",
  "requestId": "550e8400-e29b-41d4-a716-446655440000",
  "status": 200,
  "body": "{\"quoteId\":\"q_buy_abc123\",\"total\":\"512.50\",...}"
}
```

**Error:**

```json
{
  "type": "API_RESPONSE",
  "requestId": "550e8400-e29b-41d4-a716-446655440000",
  "status": 0,
  "error": "SocketException: Connection failed",
  "body": "{\"error\":\"SocketException: Connection failed\"}"
}
```

---

## Theming (`theme` object)

| Field                                              | Description                   |
| -------------------------------------------------- | ----------------------------- |
| `mode`                                             | `"light"` or `"dark"`         |
| `primaryColor`                                     | Brand primary hex             |
| `brandColor`, `brandDarkColor`, `accentColor`      | Optional palette overrides    |
| `branding.partnerName`, `logoUrl`, `walletName`, … | White-label labels and assets |

---

## TypeScript types

Import shared types from `@justgold/sdk-bridge`:

```ts
import type { SdkSessionConfig, SdkOutboundEvent, PaymentRequiredPayload } from '@justgold/sdk-bridge';
```

---

## Related

- [SDK overview](overview.md)
- [Flutter integration](flutter.md)
- [React Native integration](react-native.md)
- [Session token](session-token.md)
