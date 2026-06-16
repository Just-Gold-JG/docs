# Prices

Endpoints for fetching current and historical buy and sell prices for gold and silver.

## Latest price

Returns the current buy and sell prices for gold and silver.

#### Endpoint

```http
GET /v1/prices/latest
```

#### Authentication

This endpoint requires:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](api/authentication.md) and [Request Signing](api/request-signing.md).

#### Sample response

```json
{
  "id": "682710cc3f1b2c7a9d5e3333",
  "price24K": 557.36,
  "sellPrice24K": 535.07,
  "silverPrice": 9.3,
  "sellPriceSilver": 8.93,
  "createdAt": "2026-05-21T08:30:00.000Z",
  "updatedAt": "2026-05-21T08:30:00.000Z"
}
```

#### Response body

| Field | Type | Description |
| --- | --- | --- |
| `id` | string | Price record identifier. |
| `price24K` | number | Current buy price per gram for 24K gold. |
| `sellPrice24K` | number | Current sell price per gram for 24K gold. |
| `silverPrice` | number or null | Current buy price per gram for silver. |
| `sellPriceSilver` | number or null | Current sell price per gram for silver. |
| `createdAt` | string | ISO timestamp when this price record was created. |
| `updatedAt` | string | ISO timestamp when this price record was last updated. |

#### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Prices fetched successfully. |
| `404 Not Found` | No price data is available. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |

## Price history

Returns daily average prices for gold or silver over a selected time range. Use this to render price charts or show historical price movement to customers.

Results are cached for 24 hours.

#### Endpoint

```http
GET /v1/prices/history
```

#### Authentication

This endpoint requires:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](api/authentication.md) and [Request Signing](api/request-signing.md).

#### Query parameters

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `range` | string | No | Time range. One of `7d`, `30d`, `6m`, `1y`, `3y`, `5y`, `max`. Defaults to `30d`. |

#### Sample request

```http
GET /v1/prices/history?range=30d
```

#### Sample response

```json
[
  { "date": "17-5-2026", "avgPrice": 554.12 },
  { "date": "18-5-2026", "avgPrice": 556.80 },
  { "date": "19-5-2026", "avgPrice": 557.36 }
]
```

#### Response body

The response is an array of daily price data points, sorted chronologically.

| Field | Type | Description |
| --- | --- | --- |
| `date` | string | Date in `D-M-YYYY` format. |
| `avgPrice` | number | Closing buy price per gram for that day. |

#### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | History fetched successfully. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |
