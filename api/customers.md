# Customers

Customer-related endpoints in JustGold.

## Create customer

Creates a new customer account in JustGold.

### Endpoint

```http
POST /v1/customers
```

### Authentication

This endpoint requires:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](../authentication.md) and [Request Signing](../request-signing.md).

### Request body

Required fields:

- `nationalId`
- `idType`
- `country`
- `firstName`
- `lastName`
- `gender`

Optional fields:

- `email`
- `phone`
- `kyc`

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `nationalId` | string | Yes | Customer national ID. |
| `idType` | string | Yes | Type of identification document. |
| `country` | string | Yes | Two-letter uppercase country code. |
| `firstName` | string | Yes | Customer first name. |
| `lastName` | string | Yes | Customer last name. |
| `gender` | string | Yes | Allowed values: `Male`, `Female`, `Others`. |
| `email` | string | No | Customer email address. |
| `phone` | object | No | Customer phone details. |
| `phone.countryCode` | string | No | Phone country code. |
| `phone.number` | string | No | Phone number without country code. |
| `kyc` | object | No | KYC details for the customer. |
| `kyc.kycId` | string | No | KYC record identifier. |
| `kyc.kycStatus` | string | No | KYC status. |
| `kyc.issuingAuthority` | string | No | Authority that issued the document. |
| `kyc.issuedAt` | string | No | ISO date when the document was issued. |
| `kyc.expiresAt` | string | No | ISO date when the document expires. |
| `kyc.documentType` | string | No | Document type. |
| `kyc.documentNumber` | string | No | Document number. |

### Sample request

```json
{
  "nationalId": "784198765432109",
  "idType": "EmiratesId",
  "country": "AE",
  "firstName": "Aarav",
  "lastName": "Mehta",
  "gender": "Male",
  "email": "aarav@example.com",
  "phone": {
    "countryCode": "+971",
    "number": "501234567"
  },
  "kyc": {
    "kycId": "kyc_1001",
    "kycStatus": "Approved",
    "issuingAuthority": "ICP",
    "issuedAt": "2026-05-16T00:00:00.000Z",
    "expiresAt": "2028-05-16T00:00:00.000Z",
    "documentType": "EmiratesId",
    "documentNumber": "784198765432109"
  }
}
```

### Sample response

```json
{
  "customerId": "6818744f3f1b2c7a9d5e4321"
}
```

### Response body

| Field | Type | Description |
| --- | --- | --- |
| `customerId` | string | Created or existing customer identifier. |

### Responses

| Status | Meaning |
| --- | --- |
| `201 Created` | Customer account created successfully. |
| `200 OK` | Customer already exists and the existing `customerId` is returned. |
| `400 Bad Request` | Request payload is invalid or required fields are missing. |
| `404 Not Found` | Organization was not found. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |

## Get customer holdings

Returns the current gold and silver holdings for a customer.

### Endpoint

```http
GET /v1/customers/:nationalId/holdings
```

### Authentication

This endpoint requires:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](../authentication.md) and [Request Signing](../request-signing.md).

### Path parameters

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `nationalId` | string | Yes | Customer national ID. |

### Sample response

```json
{
  "currency": "AED",
  "gold": {
    "currentQuantity": "0.538001",
    "avgBuyPrice": "557.6200",
    "invested": "300.0000",
    "currentValue": "300.0000",
    "sellValue": "288.0026",
    "returns": "0.0000",
    "pnlPercent": "0.0000"
  },
  "silver": {
    "currentQuantity": "999.000000",
    "avgBuyPrice": "9.6900",
    "invested": "9680.3100",
    "currentValue": "9680.3100",
    "sellValue": "9290.7000",
    "returns": "0.0000",
    "pnlPercent": "0.0000"
  }
}
```

### Response body

| Field | Type | Description |
| --- | --- | --- |
| `currency` | string | Currency used for valuation. |
| `gold` | object | Gold holdings. |
| `silver` | object | Silver holdings. |
| `gold.currentQuantity`, `silver.currentQuantity` | string | Current quantity held in grams. |
| `gold.avgBuyPrice`, `silver.avgBuyPrice` | string | Average buy price per gram. |
| `gold.invested`, `silver.invested` | string | Remaining invested amount based on FIFO lots. |
| `gold.currentValue`, `silver.currentValue` | string | Current value using buy price. |
| `gold.sellValue`, `silver.sellValue` | string | Current value using sell price. |
| `gold.returns`, `silver.returns` | string | Current value minus invested amount. |
| `gold.pnlPercent`, `silver.pnlPercent` | string | Profit or loss percentage. |

### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Holdings fetched successfully. |
| `400 Bad Request` | `nationalId` is missing or invalid. |
| `404 Not Found` | Customer was not found. |
| `503 Service Unavailable` | Price data is not available. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |

## Get customer vault

Returns the customer's vault balances and sellable quantities for gold and silver.

### Endpoint

```http
GET /v1/customers/:nationalId/vault
```

Example:

```http
GET /v1/customers/abc-123/vault
```

### Authentication

This endpoint requires:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](../authentication.md) and [Request Signing](../request-signing.md).

### Path parameters

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `nationalId` | string | Yes | Customer national ID. |

### Sample response

```json
{
  "currency": "AED",
  "lockInPeriodHours": 24,
  "gold": {
    "purchased": "0.538001",
    "sold": "0.000000",
    "availableInVault": "0.538001",
    "availableToSell": "0.538001",
    "lockedQuantity": "0.000000",
    "sellValue": "288.0026"
  },
  "silver": {
    "purchased": "1000.000000",
    "sold": "1.000000",
    "availableInVault": "999.000000",
    "availableToSell": "999.000000",
    "lockedQuantity": "0.000000",
    "sellValue": "9290.7000"
  }
}
```

### Response body

| Field | Type | Description |
| --- | --- | --- |
| `currency` | string | Currency used for valuation. |
| `lockInPeriodHours` | number | Lock-in period before purchased metal is sellable. |
| `gold` | object | Gold vault balances. |
| `silver` | object | Silver vault balances. |
| `gold.purchased`, `silver.purchased` | string | Total completed buy quantity in grams. |
| `gold.sold`, `silver.sold` | string | Total sold quantity in grams, excluding pending, failed, and cancelled sells. |
| `gold.availableInVault`, `silver.availableInVault` | string | Purchased quantity minus sold quantity. |
| `gold.availableToSell`, `silver.availableToSell` | string | Quantity outside the lock-in period and available to sell. |
| `gold.lockedQuantity`, `silver.lockedQuantity` | string | Quantity still inside the lock-in period. |
| `gold.sellValue`, `silver.sellValue` | string | Sell value of the quantity available to sell. |

### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Vault balances fetched successfully. |
| `400 Bad Request` | `nationalId` is missing or invalid. |
| `404 Not Found` | Customer was not found. |
| `503 Service Unavailable` | Price data is not available. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |
