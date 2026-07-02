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

See [Authentication](api/authentication.md) and [Request Signing](api/request-signing.md).

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

---

## Update customer

Updates a customer's profile information. At least one field must be provided.

#### Endpoint

```http
PATCH /v1/customers/:customerIdentifier
```

#### Authentication

This endpoint requires:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](api/authentication.md) and [Request Signing](api/request-signing.md).

#### Path parameters

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `customerIdentifier` | string | Yes | Partner-scoped customer identifier. |

#### Request body

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `firstName` | string | No | Customer's first name. |
| `lastName` | string | No | Customer's last name. |
| `nationalId` | string | No | Customer's Emirates ID or national identity document number. May be stored in masked form (e.g. `***-****-1234567-*`). |
| `nationalIdExpiry` | string | No | Expiry date of the national identity document in `YYYY-MM-DD` format. |

At least one field must be provided.

#### Sample request

```json
{
  "firstName": "Sara",
  "lastName": "Al Mansoori",
  "nationalId": "784-1990-1234567-1",
  "nationalIdExpiry": "2030-03-15"
}
```

#### Response body

Returns the updated customer object inside a `customer` wrapper.

| Field | Type | Description |
| --- | --- | --- |
| `customer.id` | string | Customer identifier. |
| `customer.identifier` | string | Partner-scoped customer identifier. |
| `customer.firstName` | string | Updated first name. |
| `customer.lastName` | string | Updated last name. |
| `customer.nationalId` | string \| null | Emirates ID or national identity document number. May be masked. |
| `customer.nationalIdExpiry` | string \| null | Expiry date of the national identity document (`YYYY-MM-DD`). |
| `customer.email` | string \| null | Customer email, when set. |
| `customer.phone` | object \| null | Customer phone, when set. |
| `customer.gender` | string | Customer gender. |
| `customer.organizationId` | string | Organization the customer belongs to. |
| `customer.termsAccepted` | boolean | Whether the customer has accepted terms. |
| `customer.createdAt` | string | Customer creation timestamp. |
| `customer.updatedAt` | string | Customer last update timestamp. |

#### Sample response

```json
{
  "customer": {
    "id": "6818744f3f1b2c7a9d5e4321",
    "identifier": "cust-10293",
    "firstName": "Sara",
    "lastName": "Al Mansoori",
    "nationalId": "***-****-1234567-*",
    "nationalIdExpiry": "2030-03-15",
    "email": "sara@example.com",
    "phone": null,
    "gender": "Female",
    "organizationId": "6818744f3f1b2c7a9d5e9999",
    "termsAccepted": true,
    "createdAt": "2026-05-01T10:00:00.000Z",
    "updatedAt": "2026-07-01T14:30:00.000Z"
  }
}
```

#### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Customer updated successfully. |
| `400 Bad Request` | Request payload is invalid or no fields were provided. |
| `404 Not Found` | Customer was not found. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |

---

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

See [Authentication](api/authentication.md) and [Request Signing](api/request-signing.md).

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

See [Authentication](api/authentication.md) and [Request Signing](api/request-signing.md).

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

---

## Get transaction by ID

Retrieves a single transaction for a customer by its identifier.

#### Endpoint

```http
GET /v1/customers/:customerIdentifier/transactions/:transactionId
```

#### Authentication

This endpoint accepts either:

- HMAC signature (`X-Client-Id`, `X-Timestamp`, `X-Signature`) — see [Authentication](api/authentication.md) and [Request Signing](api/request-signing.md)
- Bearer token issued to the customer via the JustGold SDK

#### Path parameters

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `customerIdentifier` | string | Yes | Partner-scoped customer identifier. |
| `transactionId` | string | Yes | Transaction identifier returned when the transaction was created. |

#### Response body

| Field | Type | Description |
| --- | --- | --- |
| `id` | string | Transaction identifier. |
| `type` | string | Transaction type. One of `Buy`, `Sell`, or `Delivery`. |
| `metal` | string | `Gold` or `Silver`, when applicable. |
| `status` | string | Transaction status. One of `Pending`, `Completed`, `Failed`, or `Cancelled`. |
| `quantity` | string | Transaction quantity in grams. |
| `customerId` | string | Customer identifier. |
| `organizationIds` | array | Organization hierarchy associated with the transaction. |
| `currency` | string | Transaction currency. |
| `quotedPrice` | string | Quoted price per gram. |
| `vat` | string | VAT amount applied to the transaction. |
| `platformFee` | string | Platform fee applied to the transaction. |
| `total` | string | Transaction total before fees. |
| `grandTotal` | string | Final total including all fees. |
| `notes` | string | Notes stored with the transaction, when set. |
| `paymentReference` | string | Reference identifier for the customer's payment, when set. |
| `paymentMethod` | string | Payment method used by the customer, when set. |
| `settlement` | object | Settlement breakdown, when available. See below. |
| `createdAt` | string | Transaction creation timestamp. |
| `updatedAt` | string | Transaction last update timestamp. |

##### `settlement` object

| Field | Type | Description |
| --- | --- | --- |
| `metalMargin` | string | Margin earned on the metal spread. |
| `jgMarginPct` | number | JustGold's share of the margin as a percentage. |
| `partnerMarginPct` | number | Partner's share of the margin as a percentage. |
| `jgAmount` | string | Amount allocated to JustGold. |
| `partnerAmount` | string | Amount allocated to the partner. |
| `platformFee` | string | Platform fee collected. |
| `vatOnPlatformFee` | string | VAT applied on the platform fee. |
| `totalMintingFee` | string | Total minting fee charged across all delivery items. Present on `Delivery` transactions only. |
| `totalDeliveryFee` | string | Total delivery/service fee charged. Present on `Delivery` transactions only. |
| `jgMintingFee` | string | JustGold's share of the minting fee. Present on `Delivery` transactions only. |
| `partnerMintingFee` | string | Partner's share of the minting fee. Present on `Delivery` transactions only. |
| `vatOnMintingFee` | string | VAT applied on the total minting fee. Present on `Delivery` transactions only. |
| `vatOnDeliveryFee` | string | VAT applied on the total delivery fee. Present on `Delivery` transactions only. |
| `vatOnJGMintingFee` | string | VAT applied on JustGold's minting fee share. Present on `Delivery` transactions only. |
| `vatOnPartnerMintingFee` | string | VAT applied on the partner's minting fee share. Present on `Delivery` transactions only. |
| `vatOnJGDeliveryFee` | string | VAT applied on JustGold's delivery fee share. Present on `Delivery` transactions only. |
| `vatOnPartnerDeliveryFee` | string | VAT applied on the partner's delivery fee share. Present on `Delivery` transactions only. |
| `partnerMintingFeeMarginPct` | number | Partner's margin percentage on minting fees. Present on `Delivery` transactions only. |
| `jgMintingFeeMarginPct` | number | JustGold's margin percentage on minting fees. Present on `Delivery` transactions only. |
| `partnerDeliveryFeeMarginPct` | number | Partner's margin percentage on delivery fees. Present on `Delivery` transactions only. |
| `jgDeliveryFeeMarginPct` | number | JustGold's margin percentage on delivery fees. Present on `Delivery` transactions only. |
| `totalJG` | string | Total amount due to JustGold. |
| `totalPartner` | string | Total amount due to the partner. |
| `settlementStatus` | string | `Pending` or `Settled`. |
| `settlementDate` | string | Date settlement was processed, when set. |

#### Sample response

```json
{
  "id": "682710cc3f1b2c7a9d5e1111",
  "type": "Buy",
  "metal": "Gold",
  "status": "Completed",
  "quantity": "0.1794172488147617",
  "customerId": "6818744f3f1b2c7a9d5e4321",
  "organizationIds": [
    "6818744f3f1b2c7a9d5e9999"
  ],
  "currency": "AED",
  "quotedPrice": "557.36",
  "vat": "0.00",
  "platformFee": "2.00",
  "total": "100.00",
  "grandTotal": "102.00",
  "paymentReference": "pay_3f8a9c1b2d",
  "paymentMethod": "Card",
  "settlement": {
    "metalMargin": "5.57",
    "jgMarginPct": 40,
    "partnerMarginPct": 60,
    "jgAmount": "2.23",
    "partnerAmount": "3.34",
    "platformFee": "2.00",
    "vatOnPlatformFee": "0.00",
    "totalJG": "4.23",
    "totalPartner": "3.34",
    "settlementStatus": "Pending"
  },
  "createdAt": "2026-05-30T08:15:30.000Z",
  "updatedAt": "2026-05-30T08:17:10.000Z"
}
```

#### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Transaction retrieved successfully. |
| `400 Bad Request` | `transactionId` is invalid. |
| `401 Unauthorized` | Organization context was not found for the authenticated partner. |
| `404 Not Found` | Transaction was not found. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |

---

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

See [Authentication](api/authentication.md) and [Request Signing](api/request-signing.md).

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

See [Authentication](api/authentication.md) and [Request Signing](api/request-signing.md).

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

See [Authentication](api/authentication.md) and [Request Signing](api/request-signing.md).

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
    "redeemed": "0.000000",
    "availableInVault": "0.538001",
    "availableToSell": "0.538001",
    "lockedQuantity": "0.000000",
    "sellValue": "288.0026"
  },
  "silver": {
    "purchased": "1000.000000",
    "sold": "1.000000",
    "redeemed": "0.000000",
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
| `gold.sold`, `silver.sold` | string | Total quantity sold via SELL transactions in grams, excluding pending, failed, and cancelled transactions. |
| `gold.redeemed`, `silver.redeemed` | string | Total quantity redeemed via DELIVERY transactions in grams, excluding pending, failed, and cancelled transactions. |
| `gold.availableInVault`, `silver.availableInVault` | string | Purchased quantity minus sold quantity minus redeemed quantity. |
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

## Get customer organization

Returns the organization associated with the authenticated session. Accepts either an SDK session token or an HMAC signature.

#### Endpoint

```http
GET /v1/customers/organizations/me
```

#### Authentication

Either:
- `Authorization: Bearer <sessionToken>` — SDK session JWT
- HMAC headers (`X-Client-Id`, `X-Timestamp`, `X-Signature`) — partner backend

#### Sample response

```json
{
  "id": "682710cc3f1b2c7a9d5e1111",
  "name": "Just Gold",
  "code": "JG",
  "currency": "AED"
}
```

#### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Organization returned. |
| `401 Unauthorized` | Token or HMAC signature is missing or invalid. |