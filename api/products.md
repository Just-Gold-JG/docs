# Products

Read physical product catalog entries that can be used for delivery transactions.

!> ⚠️ Delivery is only available for UAE partners

### Authentication

These endpoints require:

- `X-Client-Id`
- `X-Timestamp`
- `X-Signature`

See [Authentication](api/authentication.md) and [Request Signing](api/request-signing.md).

## List products

Returns all products, sorted with newest products first.

#### Endpoint

```http
GET /v1/products
```

#### Response body

| Field | Type | Description |
| --- | --- | --- |
| `products` | array | Product catalog entries. |
| `products[].id` | string | Product identifier. |
| `products[].metal` | string | `Gold` or `Silver`. |
| `products[].country` | string | Product country code. |
| `products[].name` | string | Product display name. |
| `products[].brand` | string | Product brand, when available. |
| `products[].image` | string | Product image file name, when available. |
| `products[].imageUrl` | string | Full product image URL, when available. |
| `products[].inStock` | boolean | Whether the product is currently in stock. |
| `products[].weight` | number | Product metal weight. |
| `products[].units` | string | Weight unit. Allowed values: `Gram`, `Kg`, `Ounce`, `Pound`. |
| `products[].purity` | string | Product purity. |
| `products[].description` | string | Product description, when available. |
| `products[].type` | string | Product type. Allowed values: `Bar`, `Coin`, `Jewellery`, `Other`. |
| `products[].serviceCharge` | number | Service charge value. |
| `products[].serviceChargeType` | string | `Fixed` or `Percent`. |
| `products[].mintingCharge` | number | Minting charge value. |
| `products[].mintingChargeType` | string | `Fixed` or `Percent`. |
| `products[].createdAt` | string | Product creation timestamp. |
| `products[].updatedAt` | string | Product last update timestamp. |

#### Sample response

```json
{
  "products": [
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
      "updatedAt": "2026-05-30T08:15:30.000Z"
    }
  ]
}
```

#### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Products returned successfully. |
| `401 Unauthorized` | Authentication failed. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |

## Get product

Returns one product by identifier.

#### Endpoint

```http
GET /v1/products/:productId
```

#### Path parameters

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `productId` | string | Yes | Product identifier. |

#### Response body

| Field | Type | Description |
| --- | --- | --- |
| `id` | string | Product identifier. |
| `metal` | string | `Gold` or `Silver`. |
| `country` | string | Product country code. |
| `name` | string | Product display name. |
| `brand` | string | Product brand, when available. |
| `image` | string | Product image file name, when available. |
| `imageUrl` | string | Full product image URL, when available. |
| `inStock` | boolean | Whether the product is currently in stock. |
| `weight` | number | Product metal weight. |
| `units` | string | Weight unit. Allowed values: `Gram`, `Kg`, `Ounce`, `Pound`. |
| `purity` | string | Product purity. |
| `description` | string | Product description, when available. |
| `type` | string | Product type. Allowed values: `Bar`, `Coin`, `Jewellery`, `Other`. |
| `serviceCharge` | number | Service charge value. |
| `serviceChargeType` | string | `Fixed` or `Percent`. |
| `mintingCharge` | number | Minting charge value. |
| `mintingChargeType` | string | `Fixed` or `Percent`. |
| `createdAt` | string | Product creation timestamp. |
| `updatedAt` | string | Product last update timestamp. |

#### Sample response

```json
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
  "updatedAt": "2026-05-30T08:15:30.000Z"
}
```

#### Responses

| Status | Meaning |
| --- | --- |
| `200 OK` | Product returned successfully. |
| `400 Bad Request` | `productId` is invalid. |
| `401 Unauthorized` | Authentication failed. |
| `404 Not Found` | Product was not found. |
| `429 Too Many Requests` | Rate limit exceeded. Retry later. |
| `500 Internal Server Error` | An unexpected error occurred on the JustGold side. |
