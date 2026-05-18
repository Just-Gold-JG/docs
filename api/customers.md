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
- `organizationId`

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
| `organizationId` | string | Yes | JustGold organization identifier. |
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
  "organizationId": "6818744f3f1b2c7a9d5e4321",
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

### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Holdings fetched successfully. |
| `400 Bad Request` | `nationalId` is missing or invalid. |
| `404 Not Found` | Customer was not found. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |
