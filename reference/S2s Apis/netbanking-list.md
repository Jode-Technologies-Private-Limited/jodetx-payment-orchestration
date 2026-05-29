---
title: Netbanking List
excerpt: Retrieve enabled Net Banking banks and bank codes.
---
Retrieve the list of net banking banks enabled for your merchant account.

## Endpoint

```http
GET /vas/netbanking
```

## Checksum Formula

```text
mid|NETBANKING_LIST|salt
```

## Headers

| Header | Required | Description |
| --- | --- | --- |
| `X-Merchant-Id` | Yes | Merchant ID |
| `X-Checksum` | Yes | HMAC-SHA256 checksum |

## cURL

```bash
curl -X GET https://<uat-host>:8085/s2s/v1/vas/netbanking \
  -H "X-Merchant-Id: B10001" \
  -H "X-Checksum: <checksum>"
```

## Success Response

```json
{
  "banks": [
    {
      "bankCode": "HDFC",
      "bankName": "HDFC Bank",
      "status": "ACTIVE"
    },
    {
      "bankCode": "ICICI",
      "bankName": "ICICI Bank",
      "status": "ACTIVE"
    }
  ],
  "response": {
    "status": "SUCCESS",
    "statusCode": "200"
  }
}
```

## Response Fields

| Field | Type | Description |
| --- | --- | --- |
| `banks[].bankCode` | string | Bank code to pass as `paymentCode` for Net Banking |
| `banks[].bankName` | string | Customer-facing bank name |
| `banks[].status` | string | Availability status for the bank |
| `response.status` | string | Gateway status |
| `response.statusCode` | string | Gateway status code |

## Error Scenarios

| Scenario | HTTP Code | Resolution |
| --- | --- | --- |
| Invalid merchant | `401` | Verify `X-Merchant-Id` |
| Invalid checksum | `401` | Generate checksum using `mid|NETBANKING_LIST|salt` |
| Service unavailable | `500` | Retry with backoff |
Retrieve the list of net banking banks enabled for your merchant account.

## Endpoint

```http
GET /vas/netbanking
```

## Checksum Formula

```text
mid|NETBANKING_LIST|salt
```

## Headers

| Header | Required | Description |
| --- | --- | --- |
| `X-Merchant-Id` | Yes | Merchant ID |
| `X-Checksum` | Yes | HMAC-SHA256 checksum |

## cURL

```bash
curl -X GET https://<uat-host>:8085/s2s/v1/vas/netbanking \
  -H "X-Merchant-Id: B10001" \
  -H "X-Checksum: <checksum>"
```

## Success Response

```json
{
  "banks": [
    {
      "bankCode": "HDFC",
      "bankName": "HDFC Bank",
      "status": "ACTIVE"
    },
    {
      "bankCode": "ICICI",
      "bankName": "ICICI Bank",
      "status": "ACTIVE"
    }
  ],
  "response": {
    "status": "SUCCESS",
    "statusCode": "200"
  }
}
```

## Response Fields

| Field | Type | Description |
| --- | --- | --- |
| `banks[].bankCode` | string | Bank code to pass as `paymentCode` for Net Banking |
| `banks[].bankName` | string | Customer-facing bank name |
| `banks[].status` | string | Availability status for the bank |
| `response.status` | string | Gateway status |
| `response.statusCode` | string | Gateway status code |

## Error Scenarios

| Scenario | HTTP Code | Resolution |
| --- | --- | --- |
| Invalid merchant | `401` | Verify `X-Merchant-Id` |
| Invalid checksum | `401` | Generate checksum using `mid|NETBANKING_LIST|salt` |
| Service unavailable | `500` | Retry with backoff |
Retrieve the list of net banking banks enabled for your merchant account.

## Endpoint

```http
GET /vas/netbanking
```

## Checksum Formula

```text
mid|NETBANKING_LIST|salt
```

## Headers

| Header | Required | Description |
| --- | --- | --- |
| `X-Merchant-Id` | Yes | Merchant ID |
| `X-Checksum` | Yes | HMAC-SHA256 checksum |

## cURL

```bash
curl -X GET https://<uat-host>:8085/s2s/v1/vas/netbanking \
  -H "X-Merchant-Id: B10001" \
  -H "X-Checksum: <checksum>"
```

## Success Response

```json
{
  "banks": [
    {
      "bankCode": "HDFC",
      "bankName": "HDFC Bank",
      "status": "ACTIVE"
    },
    {
      "bankCode": "ICICI",
      "bankName": "ICICI Bank",
      "status": "ACTIVE"
    }
  ],
  "response": {
    "status": "SUCCESS",
    "statusCode": "200"
  }
}
```

## Response Fields

| Field | Type | Description |
| --- | --- | --- |
| `banks[].bankCode` | string | Bank code to pass as `paymentCode` for Net Banking |
| `banks[].bankName` | string | Customer-facing bank name |
| `banks[].status` | string | Availability status for the bank |
| `response.status` | string | Gateway status |
| `response.statusCode` | string | Gateway status code |

## Error Scenarios

| Scenario | HTTP Code | Resolution |
| --- | --- | --- |
| Invalid merchant | `401` | Verify `X-Merchant-Id` |
| Invalid checksum | `401` | Generate checksum using `mid|NETBANKING_LIST|salt` |
| Service unavailable | `500` | Retry with backoff |
Retrieve the list of net banking banks enabled for your merchant account.

## Endpoint

```http
GET /vas/netbanking
```

## Checksum Formula

```text
mid|NETBANKING_LIST|salt
```

## Headers

| Header | Required | Description |
| --- | --- | --- |
| `X-Merchant-Id` | Yes | Merchant ID |
| `X-Checksum` | Yes | HMAC-SHA256 checksum |

## cURL

```bash
curl -X GET https://<uat-host>:8085/s2s/v1/vas/netbanking \
  -H "X-Merchant-Id: B10001" \
  -H "X-Checksum: <checksum>"
```

## Success Response

```json
{
  "banks": [
    {
      "bankCode": "HDFC",
      "bankName": "HDFC Bank",
      "status": "ACTIVE"
    },
    {
      "bankCode": "ICICI",
      "bankName": "ICICI Bank",
      "status": "ACTIVE"
    }
  ],
  "response": {
    "status": "SUCCESS",
    "statusCode": "200"
  }
}
```

## Response Fields

| Field | Type | Description |
| --- | --- | --- |
| `banks[].bankCode` | string | Bank code to pass as `paymentCode` for Net Banking |
| `banks[].bankName` | string | Customer-facing bank name |
| `banks[].status` | string | Availability status for the bank |
| `response.status` | string | Gateway status |
| `response.statusCode` | string | Gateway status code |

## Error Scenarios

| Scenario | HTTP Code | Resolution |
| --- | --- | --- |
| Invalid merchant | `401` | Verify `X-Merchant-Id` |
| Invalid checksum | `401` | Generate checksum using `mid|NETBANKING_LIST|salt` |
| Service unavailable | `500` | Retry with backoff |