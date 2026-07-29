# Flutter integration

Use **`justgold_sdk`** on [pub.dev](https://pub.dev) to embed the JustGold trading UI in Flutter apps via **`JustGoldConnect`**.

> **New to JustGold?** Start with the unified guide: **[Partner SDK Quickstart](overview.md)** (Flutter section).

## Installation

```yaml
dependencies:
  justgold_sdk: ^1.0.1
```

```bash
flutter pub get
```

## Usage

The wrapper loads the SDK UI from JustGold CDN automatically (`GET /v1/sdk/ui-url`). **No `sdkUrl` parameter required** in normal integration.

```dart
import 'package:flutter/material.dart';
import 'package:justgold_sdk/justgold_sdk.dart';

class TradingPage extends StatefulWidget {
  const TradingPage({
    super.key,
    required this.token,
    required this.refreshToken,
    this.sandbox = false,
  });

  final String token;
  final String refreshToken;
  final bool sandbox;

  @override
  State<TradingPage> createState() => _TradingPageState();
}

class _TradingPageState extends State<TradingPage> {
  late String _token = widget.token;
  String? _refreshToken = widget.refreshToken;

  @override
  Widget build(BuildContext context) {
    return JustGoldConnect(
      token: _token,
      refreshToken: _refreshToken,
      sandbox: widget.sandbox,
      locale: 'en',
      theme: const SdkTheme(mode: SdkThemeMode.light, primaryColor: '#2563eb'),
      onClose: () => Navigator.of(context).pop(),
      onSessionExpired: _reissueSession,
      onAuthRequired: (_) => _reissueSession(),
      onTokensRefreshed: (payload) {
        setState(() {
          _token = payload['sessionToken'] as String;
          _refreshToken = payload['refreshToken'] as String?;
        });
      },
      onPaymentRequired: (payload, resume) {
        Navigator.of(context).push(
          MaterialPageRoute(builder: (_) => PartnerPaymentPage(payload: payload)),
        );
      },
      onSuccess: (payload) => debugPrint('Transaction: $payload'),
      onError: (err) => debugPrint('SDK error: $err'),
    );
  }

  Future<void> _reissueSession() async {
    final next = await partnerBackend.fetchJustGoldSession();
    if (!mounted) return;
    setState(() {
      _token = next.sessionToken;
      _refreshToken = next.refreshToken;
    });
  }
}
```

Pass `sandbox: true` for sandbox/stage API; omit or set `false` for production. The SDK resolves API and CDN URLs internally.

Token renewal is automatic (~60s before JWT expiry) when `refreshToken` is provided.

## Parameters & callbacks

| Parameter / callback | Description |
| -------------------- | ----------- |
| `token` | **Required.** Session JWT |
| `refreshToken` | Optional refresh token |
| `sandbox` | `true` → sandbox API + CDN; `false` → production |
| `sdkUrl` | Optional signed CDN URL override |
| `sdkUiSignedUrl` | Optional — from backend token response |
| `locale` | `'en'` or `'ar'` |
| `theme` | `SdkTheme` — mode, colors, branding |
| `logLevel` | `debug` \| `info` \| `warn` \| `error` |
| `platformFee` | Flat fee for previews |
| `onClose` | User closed SDK |
| `onSessionExpired` | Re-issue session |
| `onAuthRequired` | Re-issue session (`payload['reason']`) |
| `onTokensRefreshed` | Persist new refresh token |
| `onPaymentRequired` | `(payload, resume)` — open payment UI |
| `onSuccess` | `TRANSACTION_COMPLETE` payload |
| `onPlatformFeeRequest` | Return dynamic fee before preview |
| `onError` | `{ code, message }` map |
| `onLog` | Structured log map |
| `onSdkEvent` | **Catch-all** — every outbound event as `Map` |

## Bridge events & payloads

Every SDK → host event includes a **JSON payload example** and handler mapping:

→ **[Bridge reference](bridge-events.md)** — complete payload catalog + full React Native & Flutter callback examples

Quick example — payment payload shape:

```dart
onPaymentRequired: (payload, resume) {
  // payload['transactionId'], payload['type'], payload['amount'],
  // payload['currency'], payload['metal'], payload['quantity']
  Navigator.of(context).push(
    MaterialPageRoute(builder: (_) => PartnerPaymentPage(payload: payload)),
  );
},
```

Use `onSdkEvent` for analytics on any event:

```dart
onSdkEvent: (event) {
  switch (event['type']) {
    case 'NAVIGATION':
    case 'QUOTE_PREVIEWED':
    case 'TRANSACTION_CONFIRMED':
    case 'DELIVERY_COMPLETE':
    case 'SESSION_STARTED':
      analytics.log('justgold_sdk', event);
  }
},
```

## Full-page payment

When the user confirms a buy/sell, the SDK creates a **Pending** transaction and emits `PAYMENT_REQUIRED`. Your app collects payment and PATCHes transaction status via the partner HMAC API.

### Recommended flow

Keep `JustGoldConnect` **mounted**. Push a full-screen payment route on top:

```dart
onPaymentRequired: (payload, _) {
  Navigator.of(context).push(
    MaterialPageRoute(builder: (_) => PartnerPaymentPage(payload: payload)),
  );
},
// PATCH via HMAC, then Navigator.pop — SDK polls and shows the result
```

### Unmounting during payment

If you **unmount** `JustGoldConnect` during payment, remount it with the **same** `token` / `refreshToken` after PATCH. The wrapper restores the payment route via session cache.

### Optional: `resume(transactionId)`

The second callback argument posts `PAYMENT_RESULT` for instant navigation. **Not required** for the recommended overlay flow.

## Token issuance

**Production:** issue tokens from your backend using the HMAC signing algorithm (see [Backend authentication](session-token.md)). The Flutter package includes `PartnerApiClient` helpers for **development only** — do not ship `clientSecret` in mobile apps.

## Platform requirements

No extra iOS or Android configuration is needed for CDN mode. The SDK loads the signed UI URL in an internal WebView.

## Reference app

Full B2B host (token issuance, theme, async payment): [`apps/flutter-b2b`](https://github.com/Just-Gold-JG/justgold.b2b). Run with `yarn sync:postman-env`, `yarn init:flutter-b2b`, `yarn dev:flutter-b2b`.

## Next steps

→ [Bridge reference — all events with JSON payloads](bridge-events.md)  
→ [Backend authentication — session token & UI URL](session-token.md)
