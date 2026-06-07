# API Integration

Use the API integration when you want full control of the customer experience and your backend can orchestrate JustGold operations directly.

## When to use APIs

Choose APIs if you need to:

- design your own buy, sell, delivery, and portfolio screens
- run all transaction logic through your backend
- keep payment, ledger, compliance, and notification systems tightly coupled
- customize retries, reconciliation, reporting, and observability
- support web, mobile, or branch-assisted experiences from one backend

## What your system does

Your platform is responsible for:

- authenticating your customer
- creating or mapping the customer in JustGold
- calling JustGold endpoints from your secure backend
- signing requests with your `client_secret`
- verifying signed responses
- presenting quotes and confirmations to the customer
- storing transaction IDs and partner references
- handling webhook updates

## API journey

```mermaid
flowchart LR
    A[Portal access] --> B[API credentials]
    B --> C[Request signing]
    C --> D[Create or map customer]
    D --> E[Fetch price]
    E --> F[Preview order]
    F --> G[Confirm order]
    G --> H[Payment]
    H --> I[Track transaction]
    I --> J[Handle webhooks]
```

## Sequence diagram

Use this flow when your backend calls JustGold directly and your product owns the customer screens, payment flow, and transaction status experience.

### Buy sequence

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant App as Partner App
    participant Backend as Partner Backend
    participant JG as JustGold API
    participant Pay as Payment System

    rect rgb(248, 250, 252)
        Note over Customer,JG: Partner product owns account and quote screens
        Customer->>App: Open gold buy screen
        App->>Backend: Request latest quote
        Backend->>JG: POST /v1/customers if customer is not mapped
        JG-->>Backend: Customer reference
        Backend->>JG: POST /v1/buy/preview
        JG-->>Backend: Quote, price, quantity, fees, expiry
        Backend-->>App: Return quote summary
        App-->>Customer: Show quote summary
    end

    rect rgb(255, 251, 235)
        Note over Customer,Pay: Partner product owns payment collection
        Customer->>App: Confirm buy
        App->>Backend: Create pending transaction
        Backend->>JG: Create pending buy transaction
        JG-->>Backend: Pending transaction reference
        Backend-->>App: Pending transaction reference
        App->>Pay: Process payment
        Pay-->>App: Payment success
        App->>Backend: Confirm payment result
    end

    rect rgb(240, 253, 244)
        Note over Backend,App: Backend places the JustGold order and app shows final status
        Backend->>JG: Confirm buy transaction with payment reference
        JG-->>Backend: Transaction confirmed
        Backend-->>App: Transaction status
        App-->>Customer: Show transaction status
    end
```

### Sell sequence

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant App as Partner App
    participant Backend as Partner Backend
    participant JG as JustGold API

    rect rgb(248, 250, 252)
        Note over Customer,JG: Partner product owns the sell quote screen
        Customer->>App: Open gold sell screen
        App->>Backend: Request sell quote
        Backend->>JG: POST /v1/sell/preview
        JG-->>Backend: Quote, price, quantity, fees, expiry
        Backend-->>App: Return sell quote summary
        App-->>Customer: Show sell quote summary
    end

    rect rgb(240, 253, 244)
        Note over Customer,App: Sell does not require a payment handoff
        Customer->>App: Confirm sell
        App->>Backend: Confirm sell request
        Backend->>JG: POST /v1/sell with quote reference
        JG-->>Backend: Transaction created
        Backend-->>App: Transaction status
        App-->>Customer: Show transaction status
    end
```

### Delivery sequence

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant App as Partner App
    participant Backend as Partner Backend
    participant JG as JustGold API
    participant Pay as Payment System

    rect rgb(248, 250, 252)
        Customer->>App: Check out from cart
        App->>Backend: Request delivery quote
        Backend->>JG: POST /v1/delivery/preview with cart and useVault
        JG-->>Backend: Quote, vault usage, charges, expiry
        Backend-->>App: Return delivery quote summary
        App-->>Customer: Show delivery quote summary
        Customer->>App: Confirm delivery quote
    end

    rect rgb(255, 251, 235)
        Note over Customer,Pay: Partner product owns payment collection
        App->>Backend: Create pending delivery transaction
        Backend->>JG: POST /v1/delivery with quote reference
        JG-->>Backend: Pending transaction reference
        Backend-->>App: Pending transaction reference
        App->>Pay: Process payment
        Pay-->>App: Payment success
        App->>Backend: Confirm payment result
    end

    rect rgb(240, 253, 244)
        Note over Backend,App: App shows delivery transaction status after payment
        Backend-->>App: Transaction status
        App-->>Customer: Show transaction status
    end
```

## Build checklist

| Step | Guide |
| --- | --- |
| Get partner portal access | [Portal Access](../portal-access.md) |
| Configure credentials | [Authentication](../authentication.md) |
| Sign every request | [Request Signing](../request-signing.md) |
| Create or map customers | [Customers](customers.md) |
| Display live prices | [Prices](prices.md) |
| Generate quotes | [Preview](preview.md) |
| Place orders | [Buy](buy.md), [Sell](sell.md), [Delivery](delivery.md) |
| Reconcile orders | [Transactions](transactions.md), [Webhooks](../webhooks.md) |

## Endpoint map

| Area | Endpoints |
| --- | --- |
| Customers | `POST /v1/customers`, `GET /v1/customers`, `GET /v1/customers/:identifier/holdings`, `GET /v1/customers/:identifier/vault`, `GET /v1/customers/:identifier/transactions` |
| Prices | `GET /v1/prices/latest` |
| Products | `GET /v1/products`, `GET /v1/products/:productId` |
| Preview | `POST /v1/buy/preview`, `POST /v1/sell/preview`, `POST /v1/delivery/preview` |
| Orders | `POST /v1/buy`, `POST /v1/sell`, `POST /v1/delivery` |
| Transactions | `PATCH /v1/transactions/:transactionId` |

## Next step

Continue with [Authentication](../authentication.md), then implement [Request Signing](../request-signing.md).
