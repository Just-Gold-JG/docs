# SDK analytics (`Invest_*`)

Optional UI tap events for Mixpanel, Firebase, Amplitude, or any partner analytics SDK.

**Not required for trading.** Payment, session, and close still use `PAYMENT_REQUIRED`, `TRANSACTION_COMPLETE`, `CLOSE`, and the other control events.

Requires:

| Platform | Package | Version |
| --- | --- | --- |
| React Native | `@justgold/rn-sdk` | ^1.1.15 |
| Flutter | `justgold_sdk` | ^1.1.19 |
| Embedded UI | JustGold CDN | deployed with this release |

---

## Wire it up

Forward `name` as the event name and `params` as properties. Names are frozen strings — do not rewrite them.

### Mixpanel / Segment / Amplitude (React Native)

```tsx
<JustGoldConnect
  token={token}
  refreshToken={refreshToken}
  sandbox
  onAnalytics={({ name, params }) => mixpanel.track(name, params)}
/>
```

`onSdkEvent` also receives `{ type: 'ANALYTICS', payload }` if you prefer one handler.

```tsx
onSdkEvent={event => {
  if (event.type === 'ANALYTICS') mixpanel.track(event.payload.name, event.payload.params);
}}
```

Typed names:

```ts
import type { AnalyticsEventName, AnalyticsPayload } from '@justgold/rn-sdk';
```

### Flutter

```dart
JustGoldConnect(
  token: token,
  refreshToken: refreshToken,
  sandbox: true,
  onAnalytics: (event) {
    Mixpanel.track(
      event['name'] as String,
      event['params'] as Map<String, dynamic>?,
    );
  },
)
```

---

## Payload

```json
{
  "type": "ANALYTICS",
  "payload": {
    "name": "Invest_Gold_Performance",
    "params": { "range": "1W" }
  }
}
```

| Field | Type | Notes |
| --- | --- | --- |
| `name` | string | Exact `Invest_*` name from the catalog below |
| `params` | object, optional | Omitted when empty. Values are string, number, or boolean |

**No PII.** Customer identifiers, emails, and amounts are not included.

---

## How this differs from other events

| Event | Use for |
| --- | --- |
| **`ANALYTICS`** | Tap-level product analytics (`Invest_*`) |
| `NAVIGATION` | Screen / route changes (`/buy`, `/faqs`, …) |
| `CLOSE` | User dismissed the SDK (home back also sends `Invest_back` first) |
| `TRANSACTION_COMPLETE` / `onSuccess` | Buy/sell succeeded (success **Done** also sends `Invest_Buy{Metal}_Confirmed` or `Invest_Sell{Metal}_Confirmed`) |
| `PAYMENT_REQUIRED` | Host must collect payment — not analytics |

---

## Event catalog

`{Metal}` in names is always `Gold` or `Silver` (duplicated in the string, not a property).

### Home

| `name` | When | `params` |
| --- | --- | --- |
| `Invest_back` | Home header back / close | — |
| `Invest_Toggle_Gold` | Switcher or swipe to Gold | — |
| `Invest_Toggle_Silver` | Switcher or swipe to Silver | — |
| `Invest_Shortcut_BuyGold` | Buy shortcut (Gold) | — |
| `Invest_Shortcut_SellGold` | Sell shortcut (Gold) | — |
| `Invest_Shortcut_GoldTransactions` | Transactions shortcut (Gold) | — |
| `Invest_Shortcut_BuySilver` | Buy shortcut (Silver) | — |
| `Invest_Shortcut_SellSilver` | Sell shortcut (Silver) | — |
| `Invest_Shortcut_SilverTransactions` | Transactions shortcut (Silver) | — |

Home back emits **`Invest_back` then `CLOSE`**. Handle `CLOSE` to dismiss the SDK; use `Invest_back` only for analytics.

Delivery shortcut is not in this catalog.

### Performance chart

| `name` | When | `params` |
| --- | --- | --- |
| `Invest_Gold_Performance` | Gold chart range pill | `range`: `1W` \| `1M` \| `3M` \| `6M` \| `1Y` |
| `Invest_Silver_Performance` | Silver chart range pill | same |

The UI’s 7-day range is sent as **`1W`**.

### Returns calculator

| `name` | When | `params` |
| --- | --- | --- |
| `Invest_Gold_ReturnCalculator` | Open calculator (Gold) | — |
| `Invest_Gold_ReturnCalculator_Back` | Calculator back | — |
| `Invest_Gold_ReturnCalculator_Time` | Horizon or deposit mode | `mode`, `years` |
| `Invest_Gold_ReturnCalculator_InvestNow` | Invest now CTA | — |
| `Invest_Silver_ReturnCalculator` | Open calculator (Silver) | — |
| `Invest_Silver_ReturnCalculator_Back` | Calculator back | — |
| `Invest_Silver_ReturnCalculator_Time` | Horizon or deposit mode | `mode`, `years` |
| `Invest_Silver_ReturnCalculator_InvestNow` | Invest now CTA | — |

`mode` is `once` or `monthly`. `years` is a number.

### FAQ

| `name` | When | `params` |
| --- | --- | --- |
| `Invest_Gold_FAQ` | Open a FAQ (Gold selected) | — |
| `Invest_Gold_FAQ_close` | Close FAQ sheet (Gold) | — |
| `Invest_Silver_FAQ` | Open a FAQ (Silver selected) | — |
| `Invest_Silver_FAQ_close` | Close FAQ sheet (Silver) | — |

### Vault info sheets

Each metric has **Info** (icon tap), **Continue**, and **Close** (X / backdrop). Purchased is the invested-amount metric.

| Metric | Gold | Silver |
| --- | --- | --- |
| Worth now | `Invest_Gold_WorthNow_Info` / `_Continue` / `_Close` | `Invest_Silver_WorthNow_Info` / `_Continue` / `_Close` |
| Purchased | `Invest_Gold_Purchased_Info` / `_Continue` / `_Close` | `Invest_Silver_Purchased_Info` / `_Continue` / `_Close` |
| Growth | `Invest_Gold_Growth_Info` / `_Continue` / `_Close` | `Invest_Silver_Growth_Info` / `_Continue` / `_Close` |
| Sell value | `Invest_Gold_SellValue_Info` / `_Continue` / `_Close` | `Invest_Silver_SellValue_Info` / `_Continue` / `_Close` |

Exact names:

`Invest_Gold_WorthNow_Info`, `Invest_Gold_WorthNow_Continue`, `Invest_Gold_WorthNow_Close`, `Invest_Gold_Purchased_Info`, `Invest_Gold_Purchased_Continue`, `Invest_Gold_Purchased_Close`, `Invest_Gold_Growth_Info`, `Invest_Gold_Growth_Continue`, `Invest_Gold_Growth_Close`, `Invest_Gold_SellValue_Info`, `Invest_Gold_SellValue_Continue`, `Invest_Gold_SellValue_Close`, `Invest_Silver_WorthNow_Info`, `Invest_Silver_WorthNow_Continue`, `Invest_Silver_WorthNow_Close`, `Invest_Silver_Purchased_Info`, `Invest_Silver_Purchased_Continue`, `Invest_Silver_Purchased_Close`, `Invest_Silver_Growth_Info`, `Invest_Silver_Growth_Continue`, `Invest_Silver_Growth_Close`, `Invest_Silver_SellValue_Info`, `Invest_Silver_SellValue_Continue`, `Invest_Silver_SellValue_Close`.

### Buy

| `name` | When |
| --- | --- |
| `Invest_BuyGold_Continue` | Continue on buy amount (Gold) |
| `Invest_BuyGold_Confirmed` | Success **Done** (Gold) |
| `Invest_BuyGold_Invoice` | Open invoice (Gold) |
| `Invest_BuyGold_Invoice_Back` | Invoice back (Gold) |
| `Invest_BuyGold_Invoice_Share` | Invoice share (Gold) |
| `Invest_BuySilver_Continue` | Continue on buy amount (Silver) |
| `Invest_BuySilver_Confirmed` | Success **Done** (Silver) |
| `Invest_BuySilver_Invoice` | Open invoice (Silver) |
| `Invest_BuySilver_Invoice_Back` | Invoice back (Silver) |
| `Invest_BuySilver_Invoice_Share` | Invoice share (Silver) |

Continue fires when the customer taps Continue, including if validation then fails.

Success **Done** also emits `TRANSACTION_COMPLETE` / `onSuccess`. Closing the result with the header X does **not** send `*_Confirmed`.

### Sell

| `name` | When |
| --- | --- |
| `Invest_SellGold_Continue` | Continue on sell amount (Gold) |
| `Invest_SellGold_Confirmed` | Success **Done** (Gold) |
| `Invest_SellGold_Invoice` | Open invoice (Gold) |
| `Invest_SellGold_Invoice_Back` | Invoice back (Gold) |
| `Invest_SellGold_Invoice_Share` | Invoice share (Gold) |
| `Invest_SellGold_Info` | Available-to-sell banner tap (Gold) |
| `Invest_SellGold_Info_Done` | Cooldown sheet Done (Gold) — **reserved, not emitted yet** |
| `Invest_SellSilver_Continue` | Continue on sell amount (Silver) |
| `Invest_SellSilver_Confirmed` | Success **Done** (Silver) |
| `Invest_SellSilver_Invoice` | Open invoice (Silver) |
| `Invest_SellSilver_Invoice_Back` | Invoice back (Silver) |
| `Invest_SellSilver_Invoice_Share` | Invoice share (Silver) |
| `Invest_SellSilver_Info` | Available-to-sell banner tap (Silver) |
| `Invest_SellSilver_Info_Done` | Cooldown sheet Done (Silver) — **reserved, not emitted yet** |

---

## Full name list

Copy this list into Mixpanel / Firebase as the allowed event set:

```
Invest_back
Invest_Toggle_Gold
Invest_Toggle_Silver
Invest_Shortcut_BuyGold
Invest_Shortcut_SellGold
Invest_Shortcut_GoldTransactions
Invest_Shortcut_BuySilver
Invest_Shortcut_SellSilver
Invest_Shortcut_SilverTransactions
Invest_Gold_Performance
Invest_Silver_Performance
Invest_Gold_ReturnCalculator
Invest_Gold_ReturnCalculator_Back
Invest_Gold_ReturnCalculator_Time
Invest_Gold_ReturnCalculator_InvestNow
Invest_Silver_ReturnCalculator
Invest_Silver_ReturnCalculator_Back
Invest_Silver_ReturnCalculator_Time
Invest_Silver_ReturnCalculator_InvestNow
Invest_Gold_FAQ
Invest_Gold_FAQ_close
Invest_Silver_FAQ
Invest_Silver_FAQ_close
Invest_Gold_WorthNow_Info
Invest_Gold_WorthNow_Continue
Invest_Gold_WorthNow_Close
Invest_Gold_Purchased_Info
Invest_Gold_Purchased_Continue
Invest_Gold_Purchased_Close
Invest_Gold_Growth_Info
Invest_Gold_Growth_Continue
Invest_Gold_Growth_Close
Invest_Gold_SellValue_Info
Invest_Gold_SellValue_Continue
Invest_Gold_SellValue_Close
Invest_Silver_WorthNow_Info
Invest_Silver_WorthNow_Continue
Invest_Silver_WorthNow_Close
Invest_Silver_Purchased_Info
Invest_Silver_Purchased_Continue
Invest_Silver_Purchased_Close
Invest_Silver_Growth_Info
Invest_Silver_Growth_Continue
Invest_Silver_Growth_Close
Invest_Silver_SellValue_Info
Invest_Silver_SellValue_Continue
Invest_Silver_SellValue_Close
Invest_BuyGold_Continue
Invest_BuyGold_Confirmed
Invest_BuyGold_Invoice
Invest_BuyGold_Invoice_Back
Invest_BuyGold_Invoice_Share
Invest_BuySilver_Continue
Invest_BuySilver_Confirmed
Invest_BuySilver_Invoice
Invest_BuySilver_Invoice_Back
Invest_BuySilver_Invoice_Share
Invest_SellGold_Continue
Invest_SellGold_Confirmed
Invest_SellGold_Invoice
Invest_SellGold_Invoice_Back
Invest_SellGold_Invoice_Share
Invest_SellGold_Info
Invest_SellGold_Info_Done
Invest_SellSilver_Continue
Invest_SellSilver_Confirmed
Invest_SellSilver_Invoice
Invest_SellSilver_Invoice_Back
Invest_SellSilver_Invoice_Share
Invest_SellSilver_Info
Invest_SellSilver_Info_Done
```

TypeScript: `AnalyticsEventName` from `@justgold/rn-sdk` or `@justgold/sdk-bridge`.

---

## Related

- [Bridge events & payloads](sdk/bridge-events.md#analytics) — envelope and callback map
- [React Native](sdk/react-native.md) · [Flutter](sdk/flutter.md) · [Quickstart](sdk/quickstart.md)
