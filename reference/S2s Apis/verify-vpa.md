---
title: Verify VPA
excerpt: Validate a UPI VPA before initiating UPI payments.
---
Validate a customer's UPI VPA before initiating a UPI Collect payment.

## Endpoint

```http
POST /vas/verify_vpa
```

## Checksum Formula

```text
mid|VPA_VERIFY|vpa|salt
```

## Headers

| Header | Required | Description |
| --- | --- | --- |
| `Content-Type` | Yes | `application/json` |
| `X-Merchant-Id` | Yes | Merchant ID, for example `B10001` |
| `X-Checksum` | Yes | HMAC-SHA256 checksum |

## Request Body

```json
{
  "mid": "B10001",
  "vpa": "8898327678@upi"
}
```

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `mid` | string | Yes | Merchant ID assigned during onboarding |
| `vpa` | string | Yes | UPI Virtual Payment Address to validate |

## cURL

```bash
curl -X POST https://<uat-host>:8085/s2s/v1/vas/verify_vpa \
  -H "Content-Type: application/json" \
  -H "X-Merchant-Id: B10001" \
  -H "X-Checksum: <checksum>" \
  -d '{"encryptedData":"<encrypted-payload>"}'
```

## Success Response

```json
{
  "vpaData": {
    "vpa": "8898327678@upi",
    "isVPAValid": 1,
    "payerAccountName": ""
  },
  "response": {
    "status": "SUCCESS",
    "statusCode": "200"
  }
}
```

## Invalid VPA Response

```json
{
  "vpaData": {
    "vpa": "8898327678@upi",
    "isVPAValid": 0,
    "payerAccountName": ""
  },
  "response": {
    "status": "SUCCESS",
    "statusCode": "200"
  }
}
```

## Response Fields

| Field | Type | Description |
| --- | --- | --- |
| `vpaData.vpa` | string | VPA submitted for verification |
| `vpaData.isVPAValid` | integer | `1` if valid, `0` if invalid |
| `vpaData.payerAccountName` | string | Payer name when returned by the bank or UPI directory |
| `response.status` | string | Gateway processing status |
| `response.statusCode` | string | Gateway status code |

## Error Scenarios

| Scenario | HTTP Code | Resolution |
| --- | --- | --- |
| Missing VPA | `400` | Include `vpa` in the request body |
| Invalid checksum | `401` | Regenerate checksum using `mid|VPA_VERIFY|vpa|salt` |
| Gateway timeout | `500` | Retry after a short backoff |
Validate a customer's UPI VPA before initiating a UPI Collect payment.

## Endpoint

```http
POST /vas/verify_vpa
```

## Checksum Formula

```text
mid|VPA_VERIFY|vpa|salt
```

## Headers

| Header | Required | Description |
| --- | --- | --- |
| `Content-Type` | Yes | `application/json` |
| `X-Merchant-Id` | Yes | Merchant ID, for example `B10001` |
| `X-Checksum` | Yes | HMAC-SHA256 checksum |

## Request Body

```json
{
  "mid": "B10001",
  "vpa": "8898327678@upi"
}
```

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `mid` | string | Yes | Merchant ID assigned during onboarding |
| `vpa` | string | Yes | UPI Virtual Payment Address to validate |

## cURL

```bash
curl -X POST https://<uat-host>:8085/s2s/v1/vas/verify_vpa \
  -H "Content-Type: application/json" \
  -H "X-Merchant-Id: B10001" \
  -H "X-Checksum: <checksum>" \
  -d '{"encryptedData":"<encrypted-payload>"}'
```

## Success Response

```json
{
  "vpaData": {
    "vpa": "8898327678@upi",
    "isVPAValid": 1,
    "payerAccountName": ""
  },
  "response": {
    "status": "SUCCESS",
    "statusCode": "200"
  }
}
```

## Invalid VPA Response

```json
{
  "vpaData": {
    "vpa": "8898327678@upi",
    "isVPAValid": 0,
    "payerAccountName": ""
  },
  "response": {
    "status": "SUCCESS",
    "statusCode": "200"
  }
}
```

## Response Fields

| Field | Type | Description |
| --- | --- | --- |
| `vpaData.vpa` | string | VPA submitted for verification |
| `vpaData.isVPAValid` | integer | `1` if valid, `0` if invalid |
| `vpaData.payerAccountName` | string | Payer name when returned by the bank or UPI directory |
| `response.status` | string | Gateway processing status |
| `response.statusCode` | string | Gateway status code |

## Error Scenarios

| Scenario | HTTP Code | Resolution |
| --- | --- | --- |
| Missing VPA | `400` | Include `vpa` in the request body |
| Invalid checksum | `401` | Regenerate checksum using `mid|VPA_VERIFY|vpa|salt` |
| Gateway timeout | `500` | Retry after a short backoff |
Validate a customer's UPI VPA before initiating a UPI Collect payment.

## Endpoint

```http
POST /vas/verify_vpa
```

## Checksum Formula

```text
mid|VPA_VERIFY|vpa|salt
```

## Headers

| Header | Required | Description |
| --- | --- | --- |
| `Content-Type` | Yes | `application/json` |
| `X-Merchant-Id` | Yes | Merchant ID, for example `B10001` |
| `X-Checksum` | Yes | HMAC-SHA256 checksum |

## Request Body

```json
{
  "mid": "B10001",
  "vpa": "8898327678@upi"
}
```

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `mid` | string | Yes | Merchant ID assigned during onboarding |
| `vpa` | string | Yes | UPI Virtual Payment Address to validate |

## cURL

```bash
curl -X POST https://<uat-host>:8085/s2s/v1/vas/verify_vpa \
  -H "Content-Type: application/json" \
  -H "X-Merchant-Id: B10001" \
  -H "X-Checksum: <checksum>" \
  -d '{"encryptedData":"<encrypted-payload>"}'
```

## Success Response

```json
{
  "vpaData": {
    "vpa": "8898327678@upi",
    "isVPAValid": 1,
    "payerAccountName": ""
  },
  "response": {
    "status": "SUCCESS",
    "statusCode": "200"
  }
}
```

## Invalid VPA Response

```json
{
  "vpaData": {
    "vpa": "8898327678@upi",
    "isVPAValid": 0,
    "payerAccountName": ""
  },
  "response": {
    "status": "SUCCESS",
    "statusCode": "200"
  }
}
```

## Response Fields

| Field | Type | Description |
| --- | --- | --- |
| `vpaData.vpa` | string | VPA submitted for verification |
| `vpaData.isVPAValid` | integer | `1` if valid, `0` if invalid |
| `vpaData.payerAccountName` | string | Payer name when returned by the bank or UPI directory |
| `response.status` | string | Gateway processing status |
| `response.statusCode` | string | Gateway status code |

## Error Scenarios

| Scenario | HTTP Code | Resolution |
| --- | --- | --- |
| Missing VPA | `400` | Include `vpa` in the request body |
| Invalid checksum | `401` | Regenerate checksum using `mid|VPA_VERIFY|vpa|salt` |
| Gateway timeout | `500` | Retry after a short backoff |
Validate a customer's UPI VPA before initiating a UPI Collect payment.

## Endpoint

```http
POST /vas/verify_vpa
```

## Checksum Formula

```text
mid|VPA_VERIFY|vpa|salt
```

## Headers

| Header | Required | Description |
| --- | --- | --- |
| `Content-Type` | Yes | `application/json` |
| `X-Merchant-Id` | Yes | Merchant ID, for example `B10001` |
| `X-Checksum` | Yes | HMAC-SHA256 checksum |

## Request Body

```json
{
  "mid": "B10001",
  "vpa": "8898327678@upi"
}
```

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `mid` | string | Yes | Merchant ID assigned during onboarding |
| `vpa` | string | Yes | UPI Virtual Payment Address to validate |

## cURL

```bash
curl -X POST https://<uat-host>:8085/s2s/v1/vas/verify_vpa \
  -H "Content-Type: application/json" \
  -H "X-Merchant-Id: B10001" \
  -H "X-Checksum: <checksum>" \
  -d '{"encryptedData":"<encrypted-payload>"}'
```

## Success Response

```json
{
  "vpaData": {
    "vpa": "8898327678@upi",
    "isVPAValid": 1,
    "payerAccountName": ""
  },
  "response": {
    "status": "SUCCESS",
    "statusCode": "200"
  }
}
```

## Invalid VPA Response

```json
{
  "vpaData": {
    "vpa": "8898327678@upi",
    "isVPAValid": 0,
    "payerAccountName": ""
  },
  "response": {
    "status": "SUCCESS",
    "statusCode": "200"
  }
}
```

## Response Fields

| Field | Type | Description |
| --- | --- | --- |
| `vpaData.vpa` | string | VPA submitted for verification |
| `vpaData.isVPAValid` | integer | `1` if valid, `0` if invalid |
| `vpaData.payerAccountName` | string | Payer name when returned by the bank or UPI directory |
| `response.status` | string | Gateway processing status |
| `response.statusCode` | string | Gateway status code |

## Error Scenarios

| Scenario | HTTP Code | Resolution |
| --- | --- | --- |
| Missing VPA | `400` | Include `vpa` in the request body |
| Invalid checksum | `401` | Regenerate checksum using `mid|VPA_VERIFY|vpa|salt` |
| Gateway timeout | `500` | Retry after a short backoff |