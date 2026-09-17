# Bridge events & payloads

All platforms use the same JSON message envelope. Platform wrappers (`justgold_sdk`, `@justgold/rn-sdk`) translate bridge messages into typed callbacks — partners normally implement **callbacks**, not raw `postMessage`.

**Current SDK version:** React Native **1.1.12**, Flutter **1.1.16**. Tap analytics: [Analytics (`Invest_*`)](sdk/analytics.md). Invoice host fill: [Invoice share & download](sdk/invoice-handoff.md).

```json
{ "type": "EVENT_NAME", "payload": {} }
```

Some host → SDK messages omit `payload` (e.g. `PAYMENT_RESULT` uses top-level `transactionId`).

---

## Trading amounts & metal breakdown

Buy and sell involve a **single metal**. Delivery carts may include **both gold and silver**.

| Pattern | Buy / sell | Delivery |
| --- | --- | --- |
| Single metal | `metal`, `amount`, `quantity` on the payload | Top-level `metal` / `quantity` are **legacy aggregates** (first cart item metal + total grams) |
| Per-metal detail | — | `metalSummary` with lowercase keys `gold` and/or `silver` |
| Charge amount | Use `grandTotal` on `PAYMENT_REQUIRED` | Same — `metalSummary` does not replace order totals |

Each `metalSummary` entry (mirrors API `metalSummary`):

| Field | Description |
| --- | --- |
| `quantity` | Total grams of that metal in the order |
| `amount` | Metal purchase cost (`purchaseCost` from the preview API) |
| `vaultQuantityUsed` | Grams redeemed from the customer's vault (optional) |
| `quantityToPurchase` | Grams purchased at spot (optional) |
| `quotedPrice` | Buy price per gram for that metal (optional) |

On buy/sell events, the SDK always sends **both** `amount` and `quantity` (strings on fee requests, numbers on quote/confirm/complete events). If the customer entered only one value in the UI, the SDK derives the other from the live buy/sell price.

---

## Quick reference

### Host → SDK (you send — wrappers send most of these automatically)

| Event                   | Partner action?                                  | React Native / Flutter wrapper                 |
| ----------------------- | ------------------------------------------------ | ---------------------------------------------- |
| `INIT_SESSION`          | Pass props — wrapper sends                       | Automatic on load                              |
| `PAYMENT_RESULT`        | Optional fast-path after payment                 | `resume(transactionId)` in `onPaymentRequired` |
| `UPDATE_PLATFORM_FEE`   | Optional runtime fee change                      | `setPlatformFee` (ref API where exposed)       |
| `PARTNER_FEE_RESPONSE`  | Only if custom host (not using wrapper callback) | Wrapper handles via `onPartnerFeeRequest`      |
| `SET_LOG_LEVEL`         | Optional                                         | Pass `logLevel` prop or runtime message        |
| `GO_TO_ROUTE`           | Optional deep link into an SDK screen            | `goToRoute(route)` (RN ref / Flutter state)    |
| `ENSURE_SESSION`        | — (wrapper heartbeat)                            | Automatic on resume + every 30s while active   |

### SDK → Host (you receive — implement callbacks)

| Event                   | React Native callback                   | Flutter callback            | Required?                                 |
| ----------------------- | --------------------------------------- | --------------------------- | ----------------------------------------- |
| `WEBVIEW_READY`         | — (wrapper handles)                     | — (wrapper handles)         | —                                         |
| `SESSION_STARTED`       | `onSdkEvent`                            | `onSdkEvent`                | Optional analytics                        |
| `AUTH_REQUIRED`         | `onAuthRequired`                        | `onAuthRequired`            | **Yes** (re-issue session)                |
| `SESSION_EXPIRED`       | `onSessionExpired`                      | `onSessionExpired`          | **Yes**                                     |
| `TOKENS_REFRESHED`      | `onTokensRefreshed`                     | `onTokensRefreshed`         | **Recommended** (persist refresh token)   |
| `LOG`                   | `onLog`                                 | `onLog`                     | Optional                                  |
| `PARTNER_FEE_REQUEST`   | `onPartnerFeeRequest`                   | `onPartnerFeeRequest`       | If dynamic fee                            |
| `PARTNER_ACTION`        | `onPartnerAction`                       | `onPartnerAction`           | If fee dialog uses `proceed` actions      |
| `QUOTE_PREVIEWED`       | `onQuotePreviewed` / `onSdkEvent`       | `onSdkEvent`                | Optional                                  |
| `TRANSACTION_CONFIRMED` | `onTransactionConfirmed` / `onSdkEvent` | `onSdkEvent`                | Optional                                  |
| `NAVIGATION`            | `onNavigation` / `onSdkEvent`           | `onSdkEvent`                | Optional screen analytics               |
| `ANALYTICS`             | `onAnalytics` / `onSdkEvent`            | `onAnalytics` / `onSdkEvent` | Optional UI taps (`Invest_*`)          |
| `INVOICE_SHARE`         | `onInvoiceShare`                        | `onInvoiceShare`             | If host fills invoice name             |
| `INVOICE_DOWNLOAD`      | `onInvoiceDownload`                     | `onInvoiceDownload`          | If host fills invoice name             |
| `PAYMENT_REQUIRED`      | `onPaymentRequired`                     | `onPaymentRequired`         | **Yes** (payment flow)                  |
| `PAYMENT_PENDING_CLEAR` | — (wrapper internal)                    | — (wrapper internal)        | —                                         |
| `PAYMENT_DISMISSED`     | — (wrapper internal)                    | — (wrapper internal)        | —                                         |
| `TRANSACTION_COMPLETE`  | `onSuccess`                             | `onSuccess`                 | Optional                                  |
| `DELIVERY_COMPLETE`     | `onDeliveryComplete` / `onSdkEvent`     | `onSdkEvent`                | Optional                                  |
| `CLOSE`                 | `onClose`                               | `onClose`                   | **Yes**                                   |
| `ERROR`                 | `onError`                               | `onError`                   | Recommended — if `fatal`, close / show your UI |
| `OPEN_EXTERNAL_URL`     | — (wrapper: `Linking.openURL`)          | — (wrapper: `url_launcher`) | **Automatic** — custom WebView hosts only |

> **Catch-all:** `onSdkEvent` receives **every** outbound event if you prefer one handler (typed on React Native, `Map` on Flutter).

---

## React Native integration example (all callbacks)

```tsx
import { useCallback, useState } from 'react';
import { SafeAreaProvider } from 'react-native-safe-area-context';
import { JustGoldConnect, type PaymentRequiredPayload } from '@justgold/rn-sdk';

export function TradingScreen({ initialToken, initialRefreshToken, onDone }: Props) {
  const [token, setToken] = useState(initialToken);
  const [refreshToken, setRefreshToken] = useState(initialRefreshToken);

  const reissueSession = useCallback(async () => {
    const next = await partnerBackend.fetchJustGoldSession();
    setToken(next.sessionToken);
    setRefreshToken(next.refreshToken);
  }, []);

  return (
    <SafeAreaProvider>
      <JustGoldConnect
        token={token}
        refreshToken={refreshToken}
        sandbox={false}
        locale="en"
        theme={{ mode: 'light', primaryColor: '#2563eb' }}
        onClose={onDone}
        onSessionExpired={reissueSession}
        onAuthRequired={reissueSession}
        onTokensRefreshed={({ sessionToken, refreshToken: rt }) => {
          setToken(sessionToken);
          setRefreshToken(rt);
        }}
        onPaymentRequired={(payload: PaymentRequiredPayload, resume) => {
          navigation.navigate('PartnerPayment', {
            payload,
            onDone: () => {
              navigation.goBack();
              resume(payload.transactionId); // optional fast-path
            },
          });
        }}
        onPartnerFeeRequest={async payload => partnerBackend.fetchPlatformFee(payload.operation, payload.metal)}
        onAnalytics={({ name, params }) => mixpanel.track(name, params)}
        onInvoiceShare={payload => partnerFillAndShareInvoice(payload)}
        onInvoiceDownload={payload => partnerFillAndSaveInvoice(payload)}
        onSuccess={payload => console.log('Transaction complete', payload)}
        onError={err => {
          if (err.fatal) onDone();
          else console.warn(`SDK [${err.code}]:`, err.message);
        }}
        onSdkEvent={event => console.log('SDK event', event.type, event.payload)}
      />
    </SafeAreaProvider>
  );
}
```

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
        if (err['fatal'] == true) {
          Navigator.of(context).pop();
        } else {
          debugPrint('SDK error [${err['code']}]: ${err['message']}');
        }
      },

      onLog: (log) {
        debugPrint('[JustGold ${log['level']}] ${log['message']}');
      },

      onPartnerFeeRequest: (payload) async {
        return partnerBackend.fetchPlatformFee(
          operation: payload['operation'] as String,
          metal: payload['metal'] as String?,
        );
      },

      onAnalytics: (event) {
        Mixpanel.track(
          event['name'] as String,
          event['params'] as Map<String, dynamic>?,
        );
      },

      onInvoiceShare: (payload) {
        partnerFillAndShareInvoice(payload);
      },
      onInvoiceDownload: (payload) {
        partnerFillAndSaveInvoice(payload);
      },

      onSdkEvent: (event) {
        switch (event['type']) {
          case 'NAVIGATION':
            debugPrint('SDK route: ${event['payload']}');
          case 'ANALYTICS':
          case 'INVOICE_SHARE':
          case 'INVOICE_DOWNLOAD':
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
3. Partner backend `PATCH /v1/transactions/:id` (HMAC) → `Completed`, `Failed`, or `Cancelled` (user dismissed payment)
4. Partner closes payment UI (`Navigator.pop`)
5. SDK detects the status automatically — **no new JWT** unless session expired. `Completed` / `Failed` show the result screen. `Cancelled` (and `Stale`, or `Pending` older than 10 minutes) restore buy/sell with the original amount (delivery → cart).

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
| `useHostPartnerFee`          | `boolean`        | No       | `true` when `onPartnerFeeRequest` is set                |
| `useHostInvoiceActions`      | `boolean`        | No       | `true` when `onInvoiceShare` or `onInvoiceDownload` is set |
| `logLevel`                   | `string`         | No       | `debug` \| `info` \| `warn` \| `error` (default `warn`). Native API request/response console prints need `debug` **and** a debug build (`kDebugMode` / `__DEV__`) |
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
  "useHostInvoiceActions": true,
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

### `PARTNER_FEE_RESPONSE`

Reply to SDK `PARTNER_FEE_REQUEST`. **Platform wrappers send this automatically** when you implement `onPartnerFeeRequest` — you normally do not send this yourself.

The host callback may return:

- A **number** — shorthand for `{ platformFee: number }` (buy/sell only)
- A **`PartnerFeeBreakup` object** — platform fee plus optional tax and delivery splits
- **`null`** — omit override; preview API uses org default from `GET /v1/customers/organizations/me`

**Partner rejection (balance / limits):** include an `error` object on the response (or on the object returned from `onPartnerFeeRequest`). When `error` is present, the SDK **does not** call the preview API and **does not** create a transaction. With valid `error.actions`, the SDK shows a dialog; without `actions`, it blocks **silently**. Partners own localization of `title` / `description` / `action.label` (send Arabic when session `locale` is `ar`).

All monetary fields are **flat amounts in org currency** (not percentages). Omitted or `null` fields are not sent to the preview API.

#### Field reference (`PartnerFeeBreakup`)

| Field | Buy / sell | Delivery | Description |
| --- | --- | --- | --- |
| `platformFee` | Yes | Yes | Flat platform fee override for preview |
| `platformFeeTax` | Optional | Optional | Tax on platform fee |
| `mintingFee` | — | Optional | Echo of request total (auto-filled if omitted) |
| `deliveryFee` | — | Optional | Echo of request total (auto-filled if omitted) |
| `mintingFeeToJustGold` | — | Optional | Minting portion retained by JustGold |
| `mintingFeeToJustGoldTax` | — | Optional | Tax on JustGold minting portion |
| `mintingFeeToSp` | — | Optional | Minting portion to service provider |
| `mintingFeeToSpTax` | — | Optional | Tax on SP minting portion |
| `deliveryFeeToJustGold` | — | Optional | Delivery portion retained by JustGold |
| `deliveryFeeToJustGoldTax` | — | Optional | Tax on JustGold delivery portion |
| `deliveryFeeToSp` | — | Optional | Delivery portion to service provider |
| `deliveryFeeToSpTax` | — | Optional | Tax on SP delivery portion |
| `error` | Optional | Optional | **Block preview** — see [Partner rejection](#partner-rejection-balance--limits) |

#### Partner rejection (balance / limits)

Use when the partner wallet check fails **before** JustGold creates a quote or transaction — e.g. insufficient balance, daily/monthly limits, or other partner business rules.

| Field | Type | Description |
| --- | --- | --- |
| `error.code` | number | **Partner-defined** application error code (not an HTTP status). See [recommended codes](#recommended-partner-error-codes). |
| `error.title` | string (optional) | Dialog title when `actions` are present. |
| `error.description` | string (optional) | Dialog body when `actions` are present. |
| `error.actions` | array (optional) | Partner-driven CTAs. **When present (and valid), the SDK shows a dialog.** When omitted/empty/invalid, the SDK blocks preview **silently** (no modal, no confirmation navigation). |

Each action:

| Field | Type | Description |
| --- | --- | --- |
| `id` | string | Stable id (e.g. `add_funds`) — forwarded on `PARTNER_ACTION` when `outcome` is `proceed` |
| `label` | string | Button label in the SDK dialog |
| `style` | `primary` \| `secondary` \| `destructive` (optional) | Button styling |
| `outcome` | `proceed` \| `cancel` \| `close` | See below |

| `outcome` | SDK behaviour |
| --- | --- |
| `proceed` | Dismiss dialog; emit **`PARTNER_ACTION`**; stay in SDK |
| `cancel` | Dismiss dialog only; stay in SDK |
| `close` | Dismiss dialog; emit **`CLOSE`** (exit SDK) |

> **Do not use HTTP status codes** (e.g. `402`, `403`, `429`) as `error.code`. The bridge carries **partner wallet/business codes** only.

#### Recommended partner error codes

| Code | Meaning | Suggested `actions` |
| --- | --- | --- |
| `1001` | Insufficient wallet balance | ADD FUNDS (`proceed`) + Close (`cancel`) |
| `1002` | Transaction / account limit exceeded | Close (`close`) — exits SDK |
| `1003` | Account restricted (optional) | Partner-defined |

**Scenario 1 — insufficient wallet balance (SDK dialog):**

```json
{
  "type": "PARTNER_FEE_RESPONSE",
  "requestId": "550e8400-e29b-41d4-a716-446655440000",
  "platformFee": 5.0,
  "platformFeeTax": 0.25,
  "error": {
    "code": 1001,
    "title": "Insufficient balance",
    "description": "Sorry, you do not have the sufficient balance in your wallet. Please top-up your wallet and try again",
    "actions": [
      { "id": "add_funds", "label": "ADD FUNDS", "style": "primary", "outcome": "proceed" },
      { "id": "close", "label": "Close", "style": "secondary", "outcome": "cancel" }
    ]
  }
}
```

**Scenario 2 — limit exceeded (Close exits SDK):**

```json
{
  "type": "PARTNER_FEE_RESPONSE",
  "requestId": "550e8400-e29b-41d4-a716-446655440001",
  "platformFee": 5.0,
  "error": {
    "code": 1002,
    "title": "Limit exceeded",
    "description": "Sorry, you have exceeded the maximum limit of transactions for the month. Please try again at next month to complete your transfer",
    "actions": [
      { "id": "close", "label": "Close", "style": "primary", "outcome": "close" }
    ]
  }
}
```

**`PARTNER_ACTION` (SDK → host)** — emitted when the user taps a `proceed` action:

```json
{
  "type": "PARTNER_ACTION",
  "payload": {
    "context": "partner_fee",
    "actionId": "add_funds",
    "code": 1001,
    "metadata": {
      "requestId": "550e8400-e29b-41d4-a716-446655440000",
      "operation": "buy",
      "metal": "Gold",
      "amount": "500"
    }
  }
}
```

**React Native:**

```tsx
onPartnerFeeRequest={async payload => {
  const check = await partnerBackend.checkWallet(payload);
  if (check.reason === 'insufficient_balance') {
    return {
      platformFee: 5.0,
      error: {
        code: 1001,
        title: 'Insufficient balance',
        description: check.message,
        actions: [
          { id: 'add_funds', label: 'ADD FUNDS', style: 'primary', outcome: 'proceed' },
          { id: 'close', label: 'Close', style: 'secondary', outcome: 'cancel' },
        ],
      },
    };
  }
  if (check.reason === 'limit_exceeded') {
    return {
      platformFee: 5.0,
      error: {
        code: 1002,
        title: 'Limit exceeded',
        description: check.message,
        actions: [{ id: 'close', label: 'Close', style: 'primary', outcome: 'close' }],
      },
    };
  }
  return { platformFee: 5.0 };
}}
onPartnerAction={({ actionId }) => {
  if (actionId === 'add_funds') openWalletTopUp();
}}
```

**Flutter:**

```dart
onPartnerFeeRequest: (payload) async {
  final check = await partnerBackend.checkWallet(payload);
  if (check.reason == 'insufficient_balance') {
    return PartnerFeeBreakup(
      platformFee: 5.0,
      error: PartnerFeeError(
        code: 1001,
        title: 'Insufficient balance',
        description: check.message,
        actions: const [
          PartnerFeeAction(id: 'add_funds', label: 'ADD FUNDS', style: 'primary', outcome: PartnerActionOutcome.proceed),
          PartnerFeeAction(id: 'close', label: 'Close', style: 'secondary', outcome: PartnerActionOutcome.cancel),
        ],
      ),
    );
  }
  return PartnerFeeBreakup(platformFee: 5.0);
},
onPartnerAction: (payload) {
  if (payload['actionId'] == 'add_funds') openWalletTopUp();
},
```

> **Note:** Throwing from `onPartnerFeeRequest` still falls back to org default preview (legacy behaviour). To **block** the flow, return `error` — do not throw.

#### Buy — response (platform fee only)

Matching request: `operation: "buy"`, `metal: "Gold"`, `amount: "500"`.

```json
{
  "type": "PARTNER_FEE_RESPONSE",
  "requestId": "550e8400-e29b-41d4-a716-446655440000",
  "platformFee": 5.0,
  "platformFeeTax": 0.25,
  "mintingFee": null,
  "deliveryFee": null,
  "mintingFeeToJustGold": null,
  "mintingFeeToJustGoldTax": null,
  "mintingFeeToSp": null,
  "mintingFeeToSpTax": null,
  "deliveryFeeToJustGold": null,
  "deliveryFeeToJustGoldTax": null,
  "deliveryFeeToSp": null,
  "deliveryFeeToSpTax": null
}
```

Shorthand (wrapper normalizes to `{ platformFee: 5.0 }`):

```tsx
onPartnerFeeRequest={async () => 5.0}
```

Use org default (omit override on preview):

```json
{
  "type": "PARTNER_FEE_RESPONSE",
  "requestId": "550e8400-e29b-41d4-a716-446655440000",
  "platformFee": null
}
```

#### Sell — response (platform fee only)

Same shape as buy. Only `platformFee` / `platformFeeTax` are typically set; all minting/delivery fields are `null`.

```json
{
  "type": "PARTNER_FEE_RESPONSE",
  "requestId": "661f9511-f39c-52e5-b827-557766551111",
  "platformFee": 4.0,
  "platformFeeTax": 0.2,
  "mintingFee": null,
  "deliveryFee": null,
  "mintingFeeToJustGold": null,
  "mintingFeeToJustGoldTax": null,
  "mintingFeeToSp": null,
  "mintingFeeToSpTax": null,
  "deliveryFeeToJustGold": null,
  "deliveryFeeToJustGoldTax": null,
  "deliveryFeeToSp": null,
  "deliveryFeeToSpTax": null
}
```

#### Delivery — response (full breakup)

Matching request: `operation: "delivery"`, `mintingFee: 45.0`, `deliveryFee: 25.0`.

```json
{
  "type": "PARTNER_FEE_RESPONSE",
  "requestId": "772g0622-g40d-63f6-c938-668877662222",
  "platformFee": 7.5,
  "platformFeeTax": 0.38,
  "mintingFee": 45.0,
  "deliveryFee": 25.0,
  "mintingFeeToJustGold": 31.5,
  "mintingFeeToJustGoldTax": 1.58,
  "mintingFeeToSp": 13.5,
  "mintingFeeToSpTax": 0.67,
  "deliveryFeeToJustGold": 17.5,
  "deliveryFeeToJustGoldTax": 0.88,
  "deliveryFeeToSp": 7.5,
  "deliveryFeeToSpTax": 0.38
}
```

If you omit `mintingFee` / `deliveryFee` in the response, the SDK echoes the totals from the request automatically.

---

### `SET_LOG_LEVEL`

```json
{ "type": "SET_LOG_LEVEL", "level": "debug" }
```

### `GO_TO_ROUTE`

Open an allowlisted in-SDK hash route. Unknown paths are ignored.

```json
{ "type": "GO_TO_ROUTE", "route": "/delivery" }
```

React Native: `connectRef.current?.goToRoute('/delivery')`. Flutter: `connectKey.currentState?.goToRoute('/delivery')`.

### `ENSURE_SESSION`

Wrapper → WebView: silently renew if the JWT is expired or near expiry. No partner action.

```json
{ "type": "ENSURE_SESSION" }
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

**Silent renew:** The SDK refreshes the JWT with the refresh token (~60s before expiry, on resume, and every 30s while active). Idle and background → foreground should not show overlays when the refresh token is still valid. Persist `TOKENS_REFRESHED`.

**`onAuthRequired` fallback:** Fired only after silent renew fails. Mint a **new** session — the same JWT is ignored. Do not unmount the SDK on the first failure. Wrappers retry `INIT_SESSION` after ~600ms. Optional: also mint on app resume (sample apps do this). Try again keeps ignoring unchanged tokens.

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

### `PARTNER_FEE_REQUEST`

SDK needs partner fees **before** calling the preview API (`POST /v1/buy/preview`, `/v1/sell/preview`, or `/v1/delivery/preview`). Respond via `onPartnerFeeRequest`; the wrapper sends `PARTNER_FEE_RESPONSE` automatically.

| When it fires | Operation |
| --- | --- |
| User taps preview on buy screen | `buy` |
| User taps preview on sell screen | `sell` |
| User selects delivery address and previews cart | `delivery` |

Timeout: **60 seconds**. If the host does not respond, throws, or returns `null`, the SDK omits `platformFee` on preview and the API uses the org default.

#### Field reference (request payload)

| Field | Buy | Sell | Delivery | Description |
| --- | --- | --- | --- | --- |
| `requestId` | Yes | Yes | Yes | Correlate with `PARTNER_FEE_RESPONSE` |
| `operation` | `"buy"` | `"sell"` | `"delivery"` | Which preview API follows |
| `metal` | Yes | Yes | — | `"Gold"` or `"Silver"` |
| `amount` | Yes | Yes | Optional | Order amount in org currency (string). Buy/sell: always present. |
| `quantity` | Yes | Yes | — | Metal quantity in grams (string). Buy/sell: always present. |
| `mintingFee` | `null` | `null` | number | JustGold-computed minting total |
| `deliveryFee` | `null` | `null` | number | Emirate delivery fee from org settings |

For buy/sell, the SDK sends **both** `amount` and `quantity`. The customer may enter either in the UI; the SDK computes the complementary value from the live price before emitting this event.

For delivery, `metal` and per-metal amounts are not on the fee request — use `mintingFee` / `deliveryFee` totals. Per-metal breakdown appears on `QUOTE_PREVIEWED`, `TRANSACTION_CONFIRMED`, and `PAYMENT_REQUIRED` after preview.

---

#### Buy — request

Customer buying **AED 500** of gold (customer entered amount; SDK also sends computed grams):

```json
{
  "type": "PARTNER_FEE_REQUEST",
  "payload": {
    "requestId": "550e8400-e29b-41d4-a716-446655440000",
    "operation": "buy",
    "metal": "Gold",
    "amount": "500",
    "quantity": "1.666",
    "mintingFee": null,
    "deliveryFee": null
  }
}
```

Customer buying by **grams** (2.5 g silver; SDK also sends computed amount):

```json
{
  "type": "PARTNER_FEE_REQUEST",
  "payload": {
    "requestId": "550e8400-e29b-41d4-a716-446655440001",
    "operation": "buy",
    "metal": "Silver",
    "amount": "200.00",
    "quantity": "2.5",
    "mintingFee": null,
    "deliveryFee": null
  }
}
```

**Typical host response:** see [Buy — `PARTNER_FEE_RESPONSE`](#buy--response-platform-fee-only).

---

#### Sell — request

Customer selling **AED 480** equivalent of gold:

```json
{
  "type": "PARTNER_FEE_REQUEST",
  "payload": {
    "requestId": "661f9511-f39c-52e5-b827-557766551111",
    "operation": "sell",
    "metal": "Gold",
    "amount": "480",
    "quantity": "1.6",
    "mintingFee": null,
    "deliveryFee": null
  }
}
```

Customer selling by **grams**:

```json
{
  "type": "PARTNER_FEE_REQUEST",
  "payload": {
    "requestId": "661f9511-f39c-52e5-b827-557766551112",
    "operation": "sell",
    "metal": "Gold",
    "amount": "3150.00",
    "quantity": "10.5",
    "mintingFee": null,
    "deliveryFee": null
  }
}
```

**Typical host response:** see [Sell — `PARTNER_FEE_RESPONSE`](#sell--response-platform-fee-only).

---

#### Delivery — request

Fires **after the customer selects a delivery address**. Includes JustGold-computed minting and emirate delivery totals so your backend can split fees for accounting.

```json
{
  "type": "PARTNER_FEE_REQUEST",
  "payload": {
    "requestId": "772g0622-g40d-63f6-c938-668877662222",
    "operation": "delivery",
    "amount": "1200.00",
    "mintingFee": 45.0,
    "deliveryFee": 25.0
  }
}
```

No `metal` or per-metal `quantity` on delivery fee requests — the cart may contain multiple products and metals. Per-metal breakdown is on later events (`metalSummary`).

**Typical host response:** see [Delivery — `PARTNER_FEE_RESPONSE`](#delivery--response-full-breakup).

---

#### Host callback examples

**React Native:**

```tsx
onPartnerFeeRequest={async payload => {
  if (payload.operation === 'delivery') {
    return {
      platformFee: 7.5,
      platformFeeTax: 0.38,
      mintingFeeToJustGold: 31.5,
      mintingFeeToSp: 13.5,
      deliveryFeeToJustGold: 17.5,
      deliveryFeeToSp: 7.5,
    };
  }
  // buy / sell — flat platform fee or null for org default
  return await yourBackend.fetchPlatformFee(payload.operation, payload.metal);
}}
```

**Flutter:**

```dart
onPartnerFeeRequest: (payload) async {
  if (payload['operation'] == 'delivery') {
    return {
      'platformFee': 7.5,
      'platformFeeTax': 0.38,
      'mintingFeeToJustGold': 31.5,
      'mintingFeeToSp': 13.5,
      'deliveryFeeToJustGold': 17.5,
      'deliveryFeeToSp': 7.5,
    };
  }
  return yourBackend.fetchPlatformFee(
    operation: payload['operation'] as String,
    metal: payload['metal'] as String?,
  );
},
```

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
    "quantity": 1.666,
    "platformFee": 5.0,
    "currency": "AED"
  }
}
```

| Field | Buy / sell | Delivery |
| --- | --- | --- |
| `amount` | Quote grand total (includes fees) | Order `grandTotal` |
| `quantity` | Grams traded | Total grams (all metals) — legacy aggregate |
| `metalSummary` | — | Per-metal `gold` / `silver` breakdown (see [Trading amounts](#trading-amounts--metal-breakdown)) |

**Delivery example (mixed gold + silver cart):**

```json
{
  "type": "QUOTE_PREVIEWED",
  "payload": {
    "operation": "delivery",
    "quoteId": "q_del_xyz789",
    "amount": 1250.0,
    "quantity": 15.0,
    "metalSummary": {
      "gold": {
        "quantity": 10.0,
        "amount": 800.0,
        "vaultQuantityUsed": 2.0,
        "quantityToPurchase": 8.0,
        "quotedPrice": 100.0
      },
      "silver": {
        "quantity": 5.0,
        "amount": 400.0,
        "quantityToPurchase": 5.0,
        "quotedPrice": 80.0
      }
    },
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
    "quantity": 1.666,
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
    "quantity": 10.5,
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
    "quantity": 15.0,
    "currency": "AED",
    "metalSummary": {
      "gold": {
        "quantity": 10.0,
        "amount": 800.0,
        "vaultQuantityUsed": 2.0,
        "quantityToPurchase": 8.0,
        "quotedPrice": 100.0
      },
      "silver": {
        "quantity": 5.0,
        "amount": 400.0,
        "quantityToPurchase": 5.0,
        "quotedPrice": 80.0
      }
    }
  }
}
```

#### Field reference (payload)

| Field | Buy | Sell | Delivery | Description |
| --- | --- | --- | --- | --- |
| `transactionId` | Yes | Yes | Yes | Pending transaction id |
| `type` | `"buy"` | `"sell"` | `"delivery"` | Flow type |
| `status` | Yes | Yes | Yes | `"Pending"`, `"Completed"`, or `"Failed"` |
| `amount` | Yes | Yes | Yes | Grand total (includes fees and tax) |
| `quantity` | Yes | Yes | Yes | Grams traded (delivery: total grams — legacy aggregate) |
| `currency` | Yes | Yes | Yes | Org currency code |
| `metalSummary` | — | — | Yes | Per-metal `gold` / `silver` breakdown (see [Trading amounts](#trading-amounts--metal-breakdown)) |

For delivery, prefer `metalSummary` over top-level `metal` / `quantity` when the cart mixes metals.

---

### `NAVIGATION`

In-SDK route changed — useful for screen-level analytics. Prefer **`ANALYTICS`** for tap-level `Invest_*` events — [full catalog](sdk/analytics.md).

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

### `ANALYTICS`

Optional UI tap events (`Invest_*`). **Not required for trading.** Control events (`CLOSE`, `PAYMENT_REQUIRED`, `TRANSACTION_COMPLETE`) are unchanged.

```ts
onAnalytics={({ name, params }) => mixpanel.track(name, params)}
```

```dart
onAnalytics: (event) {
  Mixpanel.track(event['name'] as String, event['params'] as Map<String, dynamic>?);
},
```

```json
{
  "type": "ANALYTICS",
  "payload": {
    "name": "Invest_Gold_Performance",
    "params": { "range": "1W" }
  }
}
```

**Full event list, Mixpanel wiring, and params:** [SDK analytics (`Invest_*`)](sdk/analytics.md).

No PII. Also delivered on `onSdkEvent`.

---

### `PAYMENT_REQUIRED`

SDK created a **Pending** transaction after the customer confirmed a quote. The partner must complete payment (or payout for sell) and PATCH status via **partner backend HMAC API**.

#### Amount fields

| Field | Meaning |
| --- | --- |
| `amount` | Transaction **subtotal** excluding platform fee, tax, minting, and delivery |
| `grandTotal` | **Buy / delivery:** total to **charge** the customer. **Sell:** net **payout** to the customer after fees |
| `platformFee` / `platformFeeTax` | Platform fee and tax from partner fee breakup (when set) |
| `mintingFee` / `deliveryFee` | Totals (delivery only; `null` for buy/sell) |
| `mintingFeeToJustGold`, `mintingFeeToSp`, … | Partner accounting splits (delivery only; `null` for buy/sell) |

Fee breakup fields on `PAYMENT_REQUIRED` mirror the last `PARTNER_FEE_RESPONSE` for that session. When no dynamic fee was supplied, optional fields may be omitted or `null`.

#### Field reference (payload)

| Field | Buy | Sell | Delivery | Type |
| --- | --- | --- | --- | --- |
| `transactionId` | Yes | Yes | Yes | string |
| `type` | `"buy"` | `"sell"` | `"delivery"` | string |
| `amount` | Yes | Yes | Yes | number |
| `grandTotal` | Yes | Yes | Yes | number |
| `currency` | Yes | Yes | Yes | string |
| `metal` | Yes | Yes | Yes | `"Gold"` \| `"Silver"`. Delivery: first cart item (legacy); prefer `metalSummary`. |
| `quantity` | Yes | Yes | Yes | number (grams). Delivery: total grams (legacy); prefer `metalSummary`. |
| `metalSummary` | — | — | Yes | Per-metal `gold` / `silver` breakdown (delivery only) |
| `platformFee` | Optional | Optional | Optional | number \| null |
| `platformFeeTax` | Optional | Optional | Optional | number \| null |
| `mintingFee` | `null` | `null` | Optional | number \| null |
| `deliveryFee` | `null` | `null` | Optional | number \| null |
| `mintingFeeToJustGold` | `null` | `null` | Optional | number \| null |
| `mintingFeeToJustGoldTax` | `null` | `null` | Optional | number \| null |
| `mintingFeeToSp` | `null` | `null` | Optional | number \| null |
| `mintingFeeToSpTax` | `null` | `null` | Optional | number \| null |
| `deliveryFeeToJustGold` | `null` | `null` | Optional | number \| null |
| `deliveryFeeToJustGoldTax` | `null` | `null` | Optional | number \| null |
| `deliveryFeeToSp` | `null` | `null` | Optional | number \| null |
| `deliveryFeeToSpTax` | `null` | `null` | Optional | number \| null |

---

#### Buy — request (SDK → host)

Customer confirmed a gold buy. **Collect `grandTotal` from the customer.**

```json
{
  "type": "PAYMENT_REQUIRED",
  "payload": {
    "transactionId": "674a1b2c3d4e5f6789012345",
    "type": "buy",
    "amount": 500.0,
    "grandTotal": 505.25,
    "currency": "AED",
    "metal": "Gold",
    "quantity": 2.5,
    "platformFee": 5.0,
    "platformFeeTax": 0.25,
    "mintingFee": null,
    "deliveryFee": null,
    "mintingFeeToJustGold": null,
    "mintingFeeToJustGoldTax": null,
    "mintingFeeToSp": null,
    "mintingFeeToSpTax": null,
    "deliveryFeeToJustGold": null,
    "deliveryFeeToJustGoldTax": null,
    "deliveryFeeToSp": null,
    "deliveryFeeToSpTax": null
  }
}
```

**Partner action:** collect **AED 505.25** → backend `PATCH` with `"status": "Completed"`.

---

#### Sell — request (SDK → host)

Customer confirmed a gold sell. **`grandTotal` is the net payout** (subtotal minus platform fee and tax).

```json
{
  "type": "PAYMENT_REQUIRED",
  "payload": {
    "transactionId": "674a1b2c3d4e5f6789012346",
    "type": "sell",
    "amount": 480.0,
    "grandTotal": 474.75,
    "currency": "AED",
    "metal": "Gold",
    "quantity": 10.5,
    "platformFee": 4.0,
    "platformFeeTax": 0.2,
    "mintingFee": null,
    "deliveryFee": null,
    "mintingFeeToJustGold": null,
    "mintingFeeToJustGoldTax": null,
    "mintingFeeToSp": null,
    "mintingFeeToSpTax": null,
    "deliveryFeeToJustGold": null,
    "deliveryFeeToJustGoldTax": null,
    "deliveryFeeToSp": null,
    "deliveryFeeToSpTax": null
  }
}
```

**Partner action:** credit **AED 474.75** to the customer's wallet / bank → backend `PATCH` with `"status": "Completed"`.

---

#### Delivery — request (SDK → host)

Customer confirmed a physical delivery order. **Collect `grandTotal`** (includes minting, delivery, platform fee, and tax).

```json
{
  "type": "PAYMENT_REQUIRED",
  "payload": {
    "transactionId": "674a1b2c3d4e5f6789012347",
    "type": "delivery",
    "amount": 1200.0,
    "grandTotal": 1278.51,
    "currency": "AED",
    "metal": "Gold",
    "quantity": 15.0,
    "metalSummary": {
      "gold": {
        "quantity": 10.0,
        "amount": 800.0,
        "vaultQuantityUsed": 2.0,
        "quantityToPurchase": 8.0,
        "quotedPrice": 100.0
      },
      "silver": {
        "quantity": 5.0,
        "amount": 400.0,
        "quantityToPurchase": 5.0,
        "quotedPrice": 80.0
      }
    },
    "platformFee": 7.5,
    "platformFeeTax": 0.38,
    "mintingFee": 45.0,
    "deliveryFee": 25.0,
    "mintingFeeToJustGold": 31.5,
    "mintingFeeToJustGoldTax": 1.58,
    "mintingFeeToSp": 13.5,
    "mintingFeeToSpTax": 0.67,
    "deliveryFeeToJustGold": 17.5,
    "deliveryFeeToJustGoldTax": 0.88,
    "deliveryFeeToSp": 7.5,
    "deliveryFeeToSpTax": 0.38
  }
}
```

**Partner action:** collect **AED 1278.51** → backend `PATCH` with `"status": "Completed"`.

---

#### Partner backend response (HMAC — not from mobile SDK)

After your payment UI completes, **your backend** updates transaction status:

```http
PATCH /v1/transactions/674a1b2c3d4e5f6789012345
Content-Type: application/json
X-Client-Id: jg_partner_123
X-Timestamp: 1767225600
X-Signature: <hmac_signature>

{
  "status": "Completed",
  "paymentReference": "partner-psp-ref-12345",
  "paymentMethod": "bank_transfer"
}
```

| Status | When to send |
| --- | --- |
| `Completed` | Payment collected (buy/delivery) or payout sent (sell) |
| `Failed` | Payment or payout failed |
| `Cancelled` | User tapped Back / dismissed the payment page without paying |

Do **not** PATCH `Stale`. JustGold marks `Pending` transactions **Stale** after 10 minutes. The SDK must not keep showing the pending payment UI for `Cancelled`, `Stale`, or `Pending` older than 10 minutes — it returns to buy (subtotal `amount`) / sell (grams) / delivery cart.

The SDK polls `GET /v1/customers/:id/transactions/:id` every 2s while your payment screen is open. Close your payment UI after PATCH:

- `Completed` / `Failed` — SDK shows success or failure
- `Cancelled` — SDK restores the trading form with the amount the customer had selected

**React Native:**

```tsx
onPaymentRequired={(payload, resume) => {
  navigation.navigate('PartnerPayment', {
    chargeAmount: payload.grandTotal,
    payload,
    onDone: () => navigation.goBack(),
  });
}}
```

**Flutter:**

```dart
onPaymentRequired: (payload, resume) {
  Navigator.of(context).push(
    MaterialPageRoute(
      builder: (_) => PartnerPaymentPage(
        chargeAmount: payload.grandTotal,
        payload: payload,
      ),
    ),
  );
},
```

---

### `PAYMENT_PENDING_CLEAR`

SDK detected a payment outcome while polling (`Completed` / `Failed` / `Cancelled` / `Stale`, or client-side 10-minute pending expiry). **Wrapper internal** — clears remount recovery state. Partners do not handle this.

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
    "amount": 512.5,
    "quantity": 1.666
  }
}
```

Maps to Flutter `onSuccess` and React Native `onSuccess`. Delivery uses `DELIVERY_COMPLETE` instead — there is no `TRANSACTION_COMPLETE` for delivery.

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

Reported SDK or API error (non-auth). Hosts receive this as `onError`.

```json
{
  "type": "ERROR",
  "payload": {
    "code": "NETWORK",
    "message": "Unable to reach API",
    "fatal": false
  }
}
```

| Field | Type | Meaning |
| --- | --- | --- |
| `code` | string | Catalogue code or coarse API class (`NETWORK`, `API`, `SERVER`) |
| `message` | string | Human-readable text |
| `fatal` | boolean (optional) | `true` — SDK is unusable; show your UI or close. Omit/`false` — recoverable; log only |
| `category` | `'load' \| 'session' \| 'webview' \| 'unknown'` | Present on fatal load/crash events |
| `retryable` | boolean (optional) | Hint for host retry (fatal load errors only) |
| `httpStatus` | number (optional) | Logging only |

Existing hosts that ignore new fields keep working. Trading API error sheets stay `fatal: false`. Auth failures use `AUTH_REQUIRED`, not `ERROR`. Session-expired overlay **Close** is `onClose` only — not a fatal `ERROR`.

The wrapper still shows a short **Failed to load JustGold Connect** fallback. Partners should replace or dismiss it when `fatal` is true.

#### Host action

- **`fatal: true`** — close or hide the SDK and show your own error screen. Retry only when `retryable` is true (fresh session / remount).
- **`fatal` omitted or `false`** — log / analytics only. The SDK keeps handling the error.

```tsx
onError={err => {
  if (!err.fatal) {
    console.warn(err.code, err.message);
    return;
  }
  // SDK cannot load — partner UI or close
  navigation.goBack();
}}
```

```dart
onError: (err) {
  if (err['fatal'] != true) {
    debugPrint('SDK error: $err');
    return;
  }
  Navigator.of(context).pop();
},
```

#### Fatal codes (SDK cannot load)

| `code` | Cause | Fatal | Host action |
| --- | --- | --- | --- |
| `SDK_UI_URL_FAILED` | Token / network / `GET /v1/sdk/ui-url` failed | yes | Show partner error UI or close; retry session |
| `SDK_UI_LOAD_FAILED` | WebView / iframe failed to load CDN HTML | yes | Show partner error UI or close |
| `SDK_SSL_PINNING_FAILED` | Certificate pin mismatch | yes | Close; do not retry on the same compromised path |
| `SDK_SESSION_INIT_TIMEOUT` | HTML loaded, no `SESSION_STARTED` after init retries (~6s) | yes | Show partner error UI or close; retry |
| `SDK_WEBVIEW_PROCESS_GONE` | WebView process died and reload still failed | yes | Show partner error UI or remount SDK |
| `SDK_RENDER_CRASH` | Uncaught UI render error | yes | Show partner error UI or close |
| `SDK_UNKNOWN` | Unusable but unclassified | yes | Show partner error UI or close |

#### Not fatal

| `code` / situation | Host action |
| --- | --- |
| `NETWORK` / `API` / `SERVER` | Log / analytics; SDK shows its own sheet |
| Partner fee 1001 / 1002 | In-SDK dialog; `onPartnerAction` for ADD FUNDS |
| Lock-in / cooldown | In-SDK UX |
| `AUTH_REQUIRED` | Re-issue tokens — not an `ERROR` event |

---

### `OPEN_EXTERNAL_URL`

The SDK UI needs to open a URL **outside** the WebView — Help screen `mailto:`, `tel:`, `https://wa.me/...`.

Invoice share/download for hosts that pass `onInvoiceShare` / `onInvoiceDownload` uses [`INVOICE_SHARE` / `INVOICE_DOWNLOAD`](#invoice_share--invoice_download) instead of this event.

**React Native and Flutter wrappers handle this automatically** — no partner callback unless you use a custom WebView host.

```json
{
  "type": "OPEN_EXTERNAL_URL",
  "payload": {
    "url": "mailto:support@justgold.app?subject=Gold%20Investment%20Support"
  }
}
```

| Platform     | Wrapper behaviour                                    |
| ------------ | ---------------------------------------------------- |
| React Native | `Linking.openURL(url)`                               |
| Flutter      | `url_launcher` with `LaunchMode.externalApplication` |

Custom hosts: listen for `OPEN_EXTERNAL_URL` and delegate to native URL APIs. Do **not** load `mailto:` or `tel:` inside the WebView.

---

### `INVOICE_SHARE` / `INVOICE_DOWNLOAD`

Emitted when the host opts in (`onInvoiceShare` / `onInvoiceDownload`). The SDK ensures the invoice exists (`GET`, then `POST` generate on 404), then hands the host a short-lived presigned URL plus AcroForm field names. It does **not** preview the unfilled PDF.

Fill the PDF widget named `form.fields.customerName` (value **`customerName`**) with the customer's full name, then share or save. **Do not** put PDF bytes on the WebView message.

Copy-paste examples: **[Invoice share & download](sdk/invoice-handoff.md)**.

```json
{
  "type": "INVOICE_SHARE",
  "payload": {
    "action": "share",
    "transactionId": "674a1b2c3d4e5f6789012345",
    "fileName": "invoice-#JG1A2B3C4D.pdf",
    "mimeType": "application/pdf",
    "url": "https://s3.amazonaws.com/customer-invoices/invoice-….pdf?X-Amz-Expires=…",
    "form": {
      "type": "acroform",
      "fields": {
        "customerName": "customerName"
      }
    }
  }
}
```

`INVOICE_DOWNLOAD` is the same shape with `"action": "download"`.

Wrappers set `useHostInvoiceActions: true` on `INIT_SESSION` when either callback is provided. Other partners keep in-SDK preview and share.

---

## Full-page partner payment (recommended)

1. SDK emits `PAYMENT_REQUIRED` and navigates to an internal pending screen (polls every 2s).
2. Partner opens a **full-screen payment route on top**, keeping `JustGoldConnect` mounted.
3. Partner backend PATCHes transaction status via HMAC (`Completed`, `Failed`, or `Cancelled` if the user backs out).
4. Partner closes the payment screen — SDK shows success/failure, or restores buy/sell with the original amount on cancel.

If the partner **must unmount** the SDK during payment, remount with the **same** `token` and `refreshToken`. The wrapper adds `resumePaymentTransactionId` to the next `INIT_SESSION`. After a `Cancelled` PATCH the SDK restores the trading form, not the pending screen.

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

| Field | Description |
| --- | --- |
| `mode` | `"light"` or `"dark"` |
| `primaryColor` | Brand primary hex |
| `brandColor`, `brandDarkColor`, `accentColor`, `gradientEndColor`, `primaryLightColor`, `buyColor`, `sellColor`, `chartLineColor`, `chartTooltipColor` | Optional palette overrides |
| `textColor`, `textMutedColor`, `headingColor` | Body, secondary, and amount/label colours (defaults from `brandDarkColor`) |
| `backgroundColor`, `surfaceColor`, `borderColor` | Page, card, and border colours |
| `fontFamily` | CSS font family. SDK UI bundles **du Co Headline 16** (Light / Regular / Bold). Pass `"du Co Headline 16"` to use the bundled font; omit for SDK default (Inter). |
| `branding.partnerName` | Partner label in UI copy. When omitted, SDK uses org name from `GET /v1/customers/organizations/me` |
| `branding.walletName` | Wallet label in payment flows. Defaults to `"{partnerName} Wallet"` |
| `branding.logoUrl` | HTTPS URL for partner logo |
| `branding.icons` | Optional map of icon name → HTTPS URL to override bundled SVG icons (`back`, `help`, `copy`, `buy`, `sell`, `delivery`, etc.) |
| `branding.supportEmail` | Help screen email (default `support@justgold.app`) |
| `branding.supportPhone` | Help screen call button (default `+971 589361909`) |
| `branding.supportWhatsApp` | Help screen WhatsApp (defaults to `supportPhone`) |

---

## TypeScript types

Import shared types from `@justgold/sdk-bridge`:

```ts
import type {
  SdkSessionConfig,
  SdkOutboundEvent,
  PaymentRequiredPayload,
  PartnerFeeBreakup,
  PartnerFeeError,
  PartnerFeeAction,
  PartnerActionPayload,
  PartnerFeeRequestPayload,
  QuotePreviewedPayload,
  TransactionConfirmedPayload,
  TransactionCompletePayload,
  DeliveryMetalBridgeSummary,
  DeliveryMetalBridgeEntry,
} from '@justgold/sdk-bridge';
```

---

## Related

- [Mobile SDK Quickstart](quickstart.md)
- [SDK overview](overview.md)
- [Flutter integration](flutter.md)
- [React Native integration](react-native.md)
- [Session token](session-token.md)
