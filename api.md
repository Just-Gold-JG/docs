# API Reference

This section provides details about the **JustGold API** endpoints available for B2B partners.  
All requests must be authenticated via **hmac signature** or include a valid **JWT** with appropriate scopes.

---

## Base URL
``` https://api.justgold.me ```

---

## Authentication

- **hmac Signature**: Client must send an hmac signature in the header. 
- **JWT Scopes**: Ensure your token has the correct permissions for each endpoint. Example:  
  - `create:transaction`
  - `read:transactions`

Include the JWT in the `Authorization` header:

---

## Endpoints

### 1. Create Buy Transaction

**POST** `/v1/transactions/buy`

###### Request Body

```json
{
  "amount": 0, // either amount or quantity needed
  "quantity": 5,
  "paymentMode": "CreditCard",
  "paymentTransactionId": "txn_123456789"
}
```
###### Response

```json
{
  "id": "txn_001",
  "type": "Buy",
  "amount": 2000.25,
  "quantity": 5,
  "status": "Pending",
  "createdAt": "2025-09-08T10:30:00Z"
}
```
Required Scope: create:transaction

### 2. Create Sell Transaction

**POST** `/v1/transactions/sell`

```json
{
  "amount": 0, // amount or quantity needed
  "quantity": 2.15 // quantity in grams
}
```

###### Response
```json
{
  "id": "txn_002",
  "type": "Sell",
  "amount": 1000.25,
  "quantity": 2.15,
  "status": "Pending",
  "createdAt": "2025-09-08T10:32:00Z"
}
```
Required Scope: create:transaction


### 3. List Transactions

**GET** `/v1/transactions`

###### Query Parameters

| Name              | Type     | Required | Values                               | Description                                |
|-------------------|----------|----------|--------------------------------------|--------------------------------------------|
| `transactionType` | string   | No       | `Buy`, `Sell`                        | Filter transactions by type.               |
| `status`          | string   | No       | `Pending`, `Completed`, `Failed`     | Filter transactions by status.             |
| `offset`          | number   | No       | Any non-negative integer             | Pagination offset (default: `0`).          |
| `limit`           | number   | No       | 1–100                                | Number of results per page (default: `10`).|


Sample Request:
``` GET /transactions?transactionType=Buy&status=Completed&offset=0&limit=10 ```

###### Response
```json 
{
  "items": [
    {
      "id": "txn_001",
      "type": "Buy",
      "amount": 10000,
      "quantity": 5,
      "status": "Completed",
      "createdAt": "2025-09-08T10:30:00Z"
    },
    {
      "id": "txn_002",
      "type": "Sell",
      "amount": 5000,
      "quantity": 2,
      "status": "Failed",
      "createdAt": "2025-09-08T10:32:00Z"
    }
  ],
  "metadata": {
    "count": 10,
    "totalPages": 1,
    "currentPage": 1,
    "hasNext": false,
    "hasPrevious": false
  }
}
```