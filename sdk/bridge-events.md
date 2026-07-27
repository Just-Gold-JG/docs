# Bridge events & payloads

Complete reference for all SDK ↔ host messages: JSON payloads, Flutter callback mapping, and a full integration example.

**Full reference:** [developers.justgold.com → Bridge events](https://developers.justgold.com/06-bridge-reference) (deployed from `docs/partners/06-bridge-reference.md` in the monorepo).

---

## Events partners must implement

| Flutter callback | Event | Your action |
| ---------------- | ----- | ----------- |
| `onPaymentRequired` | `PAYMENT_REQUIRED` | Open payment UI → PATCH transaction via partner backend (HMAC) → close UI |
| `onSessionExpired` | `SESSION_EXPIRED` | Re-issue session from partner backend |
| `onAuthRequired` | `AUTH_REQUIRED` | Same as session expired |
| `onTokensRefreshed` | `TOKENS_REFRESHED` | Persist new `refreshToken` in secure storage |
| `onClose` | `CLOSE` | Dismiss SDK screen |

## Optional callbacks

| Callback | Event(s) |
| -------- | -------- |
| `onLog` | `LOG` |
| `onPlatformFeeRequest` | `PLATFORM_FEE_REQUEST` |
| `onSuccess` | `TRANSACTION_COMPLETE` |
| `onError` | `ERROR` |
| `onSdkEvent` | **All events** (analytics) |

## Example: `PAYMENT_REQUIRED` payload

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

## Example: `TOKENS_REFRESHED` payload

```json
{
  "type": "TOKENS_REFRESHED",
  "payload": {
    "sessionToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "nEwR3fR3shT0k3nAfT3rR0t4t3..."
  }
}
```

For **every** event (`INIT_SESSION`, `QUOTE_PREVIEWED`, `TRANSACTION_CONFIRMED`, `NAVIGATION`, `DELIVERY_COMPLETE`, `API_FETCH`, etc.) with copy-paste JSON and Flutter sample code, see the [full bridge reference](https://developers.justgold.com/06-bridge-reference).

---

## Related

- [Flutter integration](flutter.md)
- [Session token](session-token.md)
- [SDK overview](overview.md)
