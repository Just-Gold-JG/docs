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

All fields are required.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `firstName` | string | Yes | Customer first name. |
| `lastName` | string | Yes | Customer last name. |
| `gender` | string | Yes | Customer gender. |
| `email` | string | Yes | Customer email address. |
| `phone` | object | Yes | Customer phone details. |
| `phone.countryCode` | string | Yes | Phone country code. |
| `phone.number` | string | Yes | Phone number without country code. |

### Sample request

```json
{
  "firstName": "Aarav",
  "lastName": "Mehta",
  "gender": "male",
  "email": "aarav@example.com",
  "phone": {
    "countryCode": "+971",
    "number": "501234567"
  }
}
```

### Sample response

```json
{
  "id": "6818744f3f1b2c7a9d5e4321"
}
```

### Responses

| Status | Meaning |
| --- | --- |
| `201 Created` | Customer account created successfully. |
| `409 Conflict` | Customer account already exists. |
| `400 Bad Request` | Request payload is invalid or required fields are missing. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |

## Customer holdings

Returns the current holdings summary for gold and silver.

### Endpoint

```http
GET /v1/customers/holdings
```

### Authentication

This endpoint requires:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](../authentication.md) and [Request Signing](../request-signing.md).

### Sample response

```json
{
  "gold": {
    "currentQuantity": "0",
    "avgBuyPrice": "0",
    "invested": "0",
    "sellValue": "0",
    "currentValue": "0",
    "returns": "0",
    "pnlPercent": "0"
  },
  "silver": {
    "currentQuantity": "0",
    "avgBuyPrice": "0",
    "invested": "0",
    "sellValue": "0",
    "currentValue": "0",
    "returns": "0",
    "pnlPercent": "0"
  },
  "totals": {
    "invested": "0",
    "currentValue": "0",
    "returns": "0",
    "pnlPercent": "0",
    "sellValue": "0"
  }
}
```

### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Holdings fetched successfully. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |
