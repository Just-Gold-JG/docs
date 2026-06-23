# SDK Integration

Use the SDK integration when you want to embed JustGold flows into a mobile app with less backend and UI work than a full API integration.

## When to use an SDK

Choose an SDK if you need:

- a faster mobile rollout
- a guided JustGold experience inside your app
- fewer custom screens for gold actions
- native app support for React Native or Flutter
- a cleaner handoff between your authenticated user and JustGold flows

## Sequence diagrams

Your app starts the JustGold SDK from an authenticated user session. Your backend still owns partner credentials, customer mapping, and any server-side token or session exchange required for the SDK.

### Initialize SDK sequence

The partner backend owns credentials and session creation. The mobile app receives only the short-lived token needed to initialize the SDK.

```mermaid
sequenceDiagram
    autonumber
    actor Partner as Partner
    participant Host as Host App
    participant Backend as Partner Backend
    participant JG as JustGold API
    participant SDK as JustGold SDK

    rect rgb(248, 250, 252)
        Note over Partner,Backend: Partner prepares a secure SDK launch from their backend
        Partner->>Backend: Request SDK launch token for customer
        Backend->>JG: Create short-lived SDK token
        JG-->>Backend: SDK token and expiry
    end

    rect rgb(255, 251, 235)
        Note over Backend,SDK: The app initializes SDK without receiving partner secrets
        Backend-->>Host: Return session and refresh token
        Host->>SDK: Initialize with SDK token
        SDK->>JG: Validate token
        JG-->>SDK: Session ready
        SDK-->>Host: SDK initialized
    end
```

### Buy sequence

The customer completes quote selection inside the SDK, moves to the host app for payment, then returns to the SDK for the transaction status screen after payment is processed.

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Host as Host App
    participant SDK as JustGold SDK
    participant JG as JustGold API
    participant Pay as Payment System
    participant Backend as Partner Backend

    rect rgb(248, 250, 252)
        Note over Customer,JG: Quote and confirmation happen inside the SDK
        Customer->>Host: Open gold buy flow
        Host->>SDK: Launch buy flow
        Customer->>SDK: Enter amount or quantity
        SDK->>JG: Get buy quote
        JG-->>SDK: Quote, price, quantity, fees, expiry
        SDK-->>Customer: Show quote summary
        Customer->>SDK: Confirm quote
    end

    rect rgb(255, 251, 235)
        Note over SDK,Pay: Payment is handled by the host app
        SDK-->>Host: Hand off for payment with quote reference
        Host->>Pay: Process payment
        Pay-->>Host: Payment success
        Host->>Backend: Confirm payment result
        Backend->>JG: Confirm buy transaction with payment reference
        JG-->>Backend: Transaction confirmed
    end

    rect rgb(240, 253, 244)
        Note over Host,SDK: Customer returns to SDK for final transaction state
        Backend-->>Host: Transaction reference
        Host->>SDK: Resume status screen
        SDK->>JG: Fetch transaction status
        JG-->>SDK: Processing, completed, or failed
        SDK-->>Customer: Show transaction status
    end
```

### Sell sequence

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Host as Host App
    participant SDK as JustGold SDK
    participant JG as JustGold API
    participant Pay as Payment System
    participant Backend as Partner Backend

    rect rgb(248, 250, 252)
        Note over Customer,JG: Quote and confirmation happen inside the SDK
        Customer->>Host: Open gold sell flow
        Host->>SDK: Launch sell flow
        SDK->>JG: Get sell quote
        JG-->>SDK: Quote, price, quantity, fees, expiry
        SDK-->>Customer: Show sell quote summary
        Customer->>SDK: Confirm sell
    end

    rect rgb(255, 251, 235)
        Note over SDK,Pay: Payment is handled by the host app
        SDK-->>Host: Hand off for payment with quote reference
        Host->>Pay: Process payment
        Pay-->>Host: Payment success
        Host->>Backend: Confirm payment result
        Backend->>JG: Confirm sell transaction with payment reference
        JG-->>Backend: Transaction confirmed
    end

    rect rgb(240, 253, 244)
        Note over Host,SDK: Customer returns to SDK for final transaction state
        Backend-->>Host: Transaction reference
        Host->>SDK: Resume status screen
        SDK->>JG: Fetch transaction status
        JG-->>SDK: Processing, completed, or failed
        SDK-->>Customer: Show transaction status
    end
```

### Delivery sequence

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Host as Host App
    participant SDK as JustGold SDK
    participant JG as JustGold API
    participant Pay as Payment System
    participant Backend as Partner Backend

    rect rgb(248, 250, 252)
        Note over Customer,JG: Cart checkout and quote preview happen inside the SDK
        Customer->>Host: Open delivery flow
        Host->>SDK: Launch delivery flow
        Customer->>SDK: Review cart and delivery address
        Customer->>SDK: Select redeem from vault option
        SDK->>JG: Preview delivery quote with cart and useVault
        JG-->>SDK: Quote, vault usage, charges, expiry
        SDK-->>Customer: Show delivery quote summary
        Customer->>SDK: Confirm delivery quote
    end

    rect rgb(255, 251, 235)
        Note over SDK,Pay: Payment is handled by the host app
        SDK-->>Host: Hand off for payment with quote reference
        Host->>Pay: Process payment
        Pay-->>Host: Payment success
        Host->>Backend: Confirm payment result
        Backend->>JG: Confirm delivery transaction with payment reference
        JG-->>Backend: Transaction confirmed
    end

    rect rgb(240, 253, 244)
        Note over Host,SDK: Customer returns to SDK for delivery transaction status
        Backend-->>Host: Transaction reference
        Host->>SDK: Resume status screen
        SDK->>JG: Fetch transaction status
        JG-->>SDK: Processing, completed, or failed
        SDK-->>Customer: Show transaction status
    end
```

## SDK responsibilities

| Area          | Partner app                     | Partner backend                        | JustGold SDK                            |
| ------------- | ------------------------------- | -------------------------------------- | --------------------------------------- |
| User identity | Authenticates customer          | Maps user to JustGold customer         | Receives launch/session data            |
| Credentials   | Never stores secrets            | Stores `client_id` and `client_secret` | Uses short-lived launch/session payload |
| Experience    | Opens SDK and handles callbacks | Reconciles status                      | Presents JustGold mobile flow           |
| Updates       | Shows result state              | Handles webhooks                       | Returns completion events               |

## Platform guides

- [React Native integration](react-native.md)
- [Flutter integration](flutter.md)

## Before you build

Confirm these items with your JustGold onboarding contact:

- SDK package name and version
- environment values for sandbox and production
- session creation endpoint or backend contract
- supported mobile platforms
- callback event names and result payloads
- production launch checklist

## Next step

Choose your platform guide: [React Native](sdk/react-native.md) or [Flutter](sdk/flutter.md).
