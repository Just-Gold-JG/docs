# Customers

Customer-related endpoints in JustGold.

## Create customer

Creates a new customer account in JustGold.

This endpoint is **idempotent**: calling it again with the same `identifier` does not create a duplicate customer. Instead, the existing customer's `customerId` is returned (see [Responses](#responses)).

#### Endpoint

```http
POST /v1/customers
```

#### Authentication

This endpoint requires:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](../authentication.md) and [Request Signing](../request-signing.md).

The organization is determined from `X-Client-Id` and does not need to be sent in the request body.

#### Request body

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `identifier` | string | Yes | Customer identifier. Can be a national ID, a unique ID from the partner's system, etc. |
| `idType` | string | No | Type of identifier. One of `Custom`, `EmiratesId`, `Passport`, etc. Defaults to `Custom` if omitted. |
| `country` | string | No | Two-letter uppercase country code. |
| `firstName` | string | No | Customer first name. |
| `lastName` | string | No | Customer last name. |
| `gender` | string | No | Allowed values: `Male`, `Female`, `Others`. |
| `email` | string | No | Customer email address. |
| `phone` | object | No | Customer phone details. |
| `phone.countryCode` | string | No | Phone country code. |
| `phone.number` | string | No | Phone number without country code. |
| `kyc` | object | No | KYC details for the customer. |
| `kyc.kycId` | string | Yes, if `kyc` is provided | KYC record identifier. |
| `kyc.kycStatus` | string | Yes, if `kyc` is provided | KYC status. |
| `kyc.issuingAuthority` | string | Yes, if `kyc` is provided | Authority that issued the document. |
| `kyc.issuedAt` | string | No | ISO date when the document was issued. |
| `kyc.expiresAt` | string | No | ISO date when the document expires. |
| `kyc.documentType` | string | No | Document type. |
| `kyc.documentNumber` | string | No | Document number. |

#### Sample request

```json
{
  "identifier": "784198765432109",
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

#### Sample response

```json
{
  "customerId": "6818744f3f1b2c7a9d5e4321",
  "identifier": "784198765432109"
}
```

#### Response body

| Field | Type | Description |
| --- | --- | --- |
| `customerId` | string | Created or existing customer's internal identifier. |
| `identifier` | string | The customer identifier as originally provided in the request (without the partner's organization prefix). |

#### Responses

| Status | Meaning |
| --- | --- |
| `201 Created` | Customer account created successfully. |
| `200 OK` | A customer with this `identifier` already exists for the organization. No new customer is created; the existing `customerId` is returned. Safe to retry. |
| `400 Bad Request` | Request payload is invalid or required fields are missing. |
| `404 Not Found` | Organization was not found. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |

## Get customer holdings

Returns the current gold and silver holdings for a customer.

#### Endpoint

```http
GET /v1/customers/:identifier/holdings
```

#### Authentication

This endpoint requires:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](../authentication.md) and [Request Signing](../request-signing.md).

#### Path parameters

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `identifier` | string | Yes | Customer identifier. Can be a national ID, a unique ID from the partner's system, etc. |

#### Sample response

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

#### Response body

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

#### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Holdings fetched successfully. |
| `400 Bad Request` | `identifier` is missing or invalid. |
| `404 Not Found` | Customer was not found. |
| `503 Service Unavailable` | Price data is not available. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |

## Get customer transactions

Returns a paginated list of transactions for a customer.

#### Endpoint

```http
GET /v1/customers/:identifier/transactions
```

#### Authentication

This endpoint requires:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](../authentication.md) and [Request Signing](../request-signing.md).

#### Path parameters

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `identifier` | string | Yes | Customer identifier. Can be a national ID, a unique ID from the partner's system, etc. |

#### Query parameters

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `page` | number | No | Page number. Defaults to `1`. |
| `limit` | number | No | Number of transactions per page. Defaults to `20`. |

#### Sample response

```json
{
  "transactions": [
    {
      "id": "6a197222858e69b9e0e6d384",
      "type": "Delivery",
      "status": "Pending",
      "quantity": "0",
      "actualPrice": "0",
      "baseCurrencyActualPrice": "0",
      "createdAt": "2026-05-29T11:01:54.986Z"
    },
    {
      "id": "6a1956e9763193901f53112f",
      "type": "Buy",
      "status": "Completed",
      "metal": "Gold",
      "quantity": "0.26900039453391198307",
      "actualPrice": "557.62",
      "baseCurrencyActualPrice": "40.871934604904632153",
      "createdAt": "2026-05-29T09:05:45.327Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 11,
    "totalPages": 1
  }
}
```

#### Response body

| Field | Type | Description |
| --- | --- | --- |
| `transactions` | array | Customer transactions. |
| `transactions[].id` | string | Transaction identifier. |
| `transactions[].type` | string | Transaction type. One of `Buy`, `Sell`, or `Delivery`. |
| `transactions[].status` | string | Transaction status. One of `Pending`, `Completed`, `Failed`, or `Cancelled`. |
| `transactions[].metal` | string | `Gold` or `Silver`, when applicable. Delivery transactions may omit this field. |
| `transactions[].quantity` | string | Transaction quantity. |
| `transactions[].actualPrice` | string | Actual price per gram. |
| `transactions[].baseCurrencyActualPrice` | string | Actual transaction value in the base currency. |
| `transactions[].createdAt` | string | Transaction creation timestamp. |
| `pagination` | object | Pagination metadata. |
| `pagination.page` | number | Current page number. |
| `pagination.limit` | number | Number of transactions per page. |
| `pagination.total` | number | Total number of matching transactions. |
| `pagination.totalPages` | number | Total number of pages. |

#### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Customer transactions fetched successfully. |
| `400 Bad Request` | `identifier`, `page`, or `limit` is invalid. |
| `404 Not Found` | Customer was not found. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |

## Get customer cart

Returns the customer's cart with full product details and quantities.

#### Endpoint

```http
GET /v1/customers/:identifier/cart
```

#### Authentication

This endpoint requires:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](../authentication.md) and [Request Signing](../request-signing.md).

#### Path parameters

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `identifier` | string | Yes | Customer identifier. Can be a national ID, a unique ID from the partner's system, etc. |

#### Sample response

```json
{
  "items": [
    {
      "id": "682710cc3f1b2c7a9d5e3333",
      "metal": "Gold",
      "country": "AE",
      "name": "Gold Bar 1g",
      "brand": "JustGold",
      "image": "gold-bar-1g.png",
      "imageUrl": "https://cdn.example.com/products/images/gold-bar-1g.png",
      "inStock": true,
      "weight": 1,
      "units": "Gram",
      "purity": "999.9",
      "description": "1 gram gold bar",
      "type": "Bar",
      "serviceCharge": 5,
      "serviceChargeType": "Fixed",
      "mintingCharge": 15,
      "mintingChargeType": "Fixed",
      "createdAt": "2026-05-30T08:15:30.000Z",
      "updatedAt": "2026-05-30T08:15:30.000Z",
      "quantity": 2
    }
  ]
}
```

#### Response body

| Field | Type | Description |
| --- | --- | --- |
| `items` | array | Cart items, each combining product details with the cart quantity. |
| `items[].id` | string | Product identifier. |
| `items[].metal` | string | `Gold` or `Silver`. |
| `items[].country` | string | Product country code. |
| `items[].name` | string | Product display name. |
| `items[].brand` | string | Product brand, when available. |
| `items[].image` | string | Product image file name, when available. |
| `items[].imageUrl` | string | Full product image URL, when available. |
| `items[].inStock` | boolean | Whether the product is currently in stock. |
| `items[].weight` | number | Product metal weight. |
| `items[].units` | string | Weight unit. Allowed values: `Gram`, `Kg`, `Ounce`, `Pound`. |
| `items[].purity` | string | Product purity. |
| `items[].description` | string | Product description, when available. |
| `items[].type` | string | Product type. Allowed values: `Bar`, `Coin`, `Jewellery`, `Other`. |
| `items[].serviceCharge` | number | Service charge value. |
| `items[].serviceChargeType` | string | `Fixed` or `Percent`. |
| `items[].mintingCharge` | number | Minting charge value. |
| `items[].mintingChargeType` | string | `Fixed` or `Percent`. |
| `items[].createdAt` | string | Product creation timestamp. |
| `items[].updatedAt` | string | Product last update timestamp. |
| `items[].quantity` | number | Quantity of this product in the cart. |

#### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Cart fetched successfully. |
| `400 Bad Request` | `identifier` is missing or invalid. |
| `404 Not Found` | Customer was not found. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |

## Update customer cart

Adds, updates, or removes a product in the customer's cart, and returns the resulting cart.

#### Endpoint

```http
PUT /v1/customers/:identifier/cart
```

#### Authentication

This endpoint requires:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](../authentication.md) and [Request Signing](../request-signing.md).

#### Path parameters

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `identifier` | string | Yes | Customer identifier. Can be a national ID, a unique ID from the partner's system, etc. |

#### Request body

Required fields:

- `productId`
- `quantity`

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `productId` | string | Yes | Identifier of the product to add, update, or remove. |
| `quantity` | number | Yes | Desired quantity for the product. Integer, `0` or greater. Setting it to `0` removes the product from the cart. Setting it for an existing product replaces (does not increment) its quantity. |

#### Sample request

```json
{
  "productId": "682710cc3f1b2c7a9d5e3333",
  "quantity": 2
}
```

#### Sample response

```json
{
  "items": [
    {
      "id": "682710cc3f1b2c7a9d5e3333",
      "metal": "Gold",
      "country": "AE",
      "name": "Gold Bar 1g",
      "brand": "JustGold",
      "image": "gold-bar-1g.png",
      "imageUrl": "https://cdn.example.com/products/images/gold-bar-1g.png",
      "inStock": true,
      "weight": 1,
      "units": "Gram",
      "purity": "999.9",
      "description": "1 gram gold bar",
      "type": "Bar",
      "serviceCharge": 5,
      "serviceChargeType": "Fixed",
      "mintingCharge": 15,
      "mintingChargeType": "Fixed",
      "createdAt": "2026-05-30T08:15:30.000Z",
      "updatedAt": "2026-05-30T08:15:30.000Z",
      "quantity": 2
    }
  ]
}
```

#### Response body

Same shape as [Get customer cart](#get-customer-cart):

| Field | Type | Description |
| --- | --- | --- |
| `items` | array | Cart items, each combining product details with the cart quantity, after applying the update. |

#### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Cart updated and returned successfully. |
| `400 Bad Request` | `identifier` is missing, or `productId`/`quantity` is missing or invalid. |
| `404 Not Found` | Customer or product was not found. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |

## Get customer token

Issues a short-lived token for a customer, used to initialize the SDK on their behalf.

#### Endpoint

```http
POST /v1/customers/:identifier/token
```

#### Authentication

This endpoint requires:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](../authentication.md) and [Request Signing](../request-signing.md).

#### Path parameters

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `identifier` | string | Yes | Customer identifier. Can be a national ID, a unique ID from the partner's system, etc. |

#### Sample response

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjdXN0b21lcklkIjoiNjgxODc0NGYzZjFiMmM3YTlkNWU0MzIxIiwiaWRlbnRpZmllciI6Ijc4NDE5ODc2NTQzMjEwOSIsIm9yZ2FuaXphdGlvbklkIjoiNjgxODc0NGYzZjFiMmM3YTlkNWUwMDAxIn0.signature"
}
```

#### Response body

| Field | Type | Description |
| --- | --- | --- |
| `token` | string | Short-lived JWT (valid for 10 minutes) identifying the customer. Use it to initialize the SDK. |

#### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Token issued successfully. |
| `400 Bad Request` | `identifier` is missing or invalid. |
| `403 Forbidden` | Customer does not belong to the requesting organization. |
| `404 Not Found` | Customer was not found. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |

## Get customer vault

Returns the customer's vault balances and sellable quantities for gold and silver.

#### Endpoint

```http
GET /v1/customers/:identifier/vault
```

Example:

```http
GET /v1/customers/abc-123/vault
```

#### Authentication

This endpoint requires:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](../authentication.md) and [Request Signing](../request-signing.md).

#### Path parameters

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `identifier` | string | Yes | Customer identifier. Can be a national ID, a unique ID from the partner's system, etc. |

#### Sample response

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

#### Response body

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

#### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Vault balances fetched successfully. |
| `400 Bad Request` | `identifier` is missing or invalid. |
| `404 Not Found` | Customer was not found. |
| `503 Service Unavailable` | Price data is not available. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |
