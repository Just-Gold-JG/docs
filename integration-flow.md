# Integration Flow

This is the recommended partner integration sequence.

## Buy flow

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Partner as Partner Platform
    participant JustGold as JustGold API

    Customer->>Partner: Create account
    Partner->>JustGold: POST /v1/customers
    JustGold-->>Partner: customerId
    Partner->>Partner: Verify response signature
    Partner-->>Customer: Account ready

    Customer->>Partner: Check gold prices
    Partner->>JustGold: GET /v1/prices
    JustGold-->>Partner: buyPrice, sellPrice
    Partner->>Partner: Verify response signature
    Partner-->>Customer: Show latest prices

    Customer->>Partner: Preview buy request
    Partner->>JustGold: POST /v1/buy/preview
    JustGold-->>Partner: quoteId
    Partner->>Partner: Verify response signature
    Partner-->>Customer: Show buy preview

    Customer->>Partner: Confirm buy order
    Partner->>JustGold: POST /v1/buy
    JustGold-->>Partner: orderId, status
    Partner->>Partner: Verify response signature
    Partner-->>Customer: Order placed
```

## Sell flow

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Partner as Partner Platform
    participant JustGold as JustGold API

    Customer->>Partner: Check gold prices
    Partner->>JustGold: GET /v1/prices
    JustGold-->>Partner: buyPrice, sellPrice
    Partner->>Partner: Verify response signature
    Partner-->>Customer: Show latest prices

    Customer->>Partner: Preview sell request
    Partner->>JustGold: POST /v1/sell/preview
    JustGold-->>Partner: quoteId
    Partner->>Partner: Verify response signature
    Partner-->>Customer: Show sell preview

    Customer->>Partner: Confirm sell order
    Partner->>JustGold: POST /v1/sell
    JustGold-->>Partner: orderId, status
    Partner->>Partner: Verify response signature
    Partner-->>Customer: Order placed
```

## 1. Create or map your customer

Use `POST /v1/customers` to register the customer in JustGold or link your internal customer record to a JustGold customer ID.

## 2. Fetch the latest prices

Use `GET /v1/prices` to display the current buy and sell prices before asking the customer to confirm a transaction.

## 3. Preview the order

Use:

- `POST /v1/buy/preview` for buy quotes
- `POST /v1/sell/preview` for sell quotes

to generate a quote before placing a final order.

## 4. Place the order

Call:

- `POST /v1/buy` for a buy order
- `POST /v1/sell` for a sell order

Use the preview response to keep the confirmed order aligned with the quoted values.

## 5. Store identifiers

Persist the following identifiers in your system:

- JustGold customer ID
- Preview ID, if returned
- Buy or sell order ID
- Your own internal transaction reference

## 6. Verify signed responses

Before using any JustGold response, verify the response signature using the shared `client_secret`.
