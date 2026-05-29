---
title: Enabled Instruments
excerpt: Retrieve all payment instruments enabled for a merchant account.
---
List all payment instruments enabled for your merchant account.

## Endpoint

```http
GET /vas/instruments
```

## Checksum Formula

```text
mid|ENABLED_INSTRUMENTS|salt
```

## Headers

| Header | Required | Description |
| --- | --- | --- |
| `X-Merchant-Id` | Yes | Merchant ID |
| `X-Checksum` | Yes | HMAC-SHA256 checksum |

## cURL

```bash
curl -X GET https://<uat-host>:8085/s2s/v1/vas/instruments \
  -H "X-Merchant-Id: B10001" \
  -H "X-Checksum: <checksum>"
```

## Success Response

```json
{
  "instruments": [
    { "paymentMode": "UPI", "paymentCode": "UPII", "name": "UPI Collect", "status": "ACTIVE" },
    { "paymentMode": "UPI", "paymentCode": "UPIC", "name": "UPI Intent", "status": "ACTIVE" },
    { "paymentMode": "UPI", "paymentCode": "DQR", "name": "Dynamic QR", "status": "ACTIVE" },
    { "paymentMode": "NB", "paymentCode": "HDFC", "name": "HDFC Bank Net Banking", "status": "ACTIVE" },
    { "paymentMode": "CC", "paymentCode": "CC", "name": "Credit Card", "status": "ACTIVE" },
    { "paymentMode": "DC", "paymentCode": "DC", "name": "Debit Card", "status": "ACTIVE" }
  ],
  "response": { "status": "SUCCESS", "statusCode": "200" }
}
```

## Response Fields

| Field | Type | Description |
| --- | --- | --- |
| `instruments[].paymentMode` | string | Payment mode to send in Initiate Transaction |
| `instruments[].paymentCode` | string | Payment code to send in Initiate Transaction |
| `instruments[].name` | string | Display name |
| `instruments[].status` | string | Instrument availability |

## Error Scenarios

| Scenario | HTTP Code | Resolution |
| --- | --- | --- |
| Checksum mismatch | `401` | Use `mid|ENABLED_INSTRUMENTS|salt` |
| Merchant not enabled | `401` | Confirm merchant activation |
| Gateway error | `500` | Retry with backoff |
List all payment instruments enabled for your merchant account.

## Endpoint

```http
GET /vas/instruments
```

## Checksum Formula

```text
mid|ENABLED_INSTRUMENTS|salt
```

## Headers

| Header | Required | Description |
| --- | --- | --- |
| `X-Merchant-Id` | Yes | Merchant ID |
| `X-Checksum` | Yes | HMAC-SHA256 checksum |

## cURL

```bash
curl -X GET https://<uat-host>:8085/s2s/v1/vas/instruments \
  -H "X-Merchant-Id: B10001" \
  -H "X-Checksum: <checksum>"
```

## Success Response

```json
{
  "instruments": [
    { "paymentMode": "UPI", "paymentCode": "UPII", "name": "UPI Collect", "status": "ACTIVE" },
    { "paymentMode": "UPI", "paymentCode": "UPIC", "name": "UPI Intent", "status": "ACTIVE" },
    { "paymentMode": "UPI", "paymentCode": "DQR", "name": "Dynamic QR", "status": "ACTIVE" },
    { "paymentMode": "NB", "paymentCode": "HDFC", "name": "HDFC Bank Net Banking", "status": "ACTIVE" },
    { "paymentMode": "CC", "paymentCode": "CC", "name": "Credit Card", "status": "ACTIVE" },
    { "paymentMode": "DC", "paymentCode": "DC", "name": "Debit Card", "status": "ACTIVE" }
  ],
  "response": { "status": "SUCCESS", "statusCode": "200" }
}
```

## Response Fields

| Field | Type | Description |
| --- | --- | --- |
| `instruments[].paymentMode` | string | Payment mode to send in Initiate Transaction |
| `instruments[].paymentCode` | string | Payment code to send in Initiate Transaction |
| `instruments[].name` | string | Display name |
| `instruments[].status` | string | Instrument availability |

## Error Scenarios

| Scenario | HTTP Code | Resolution |
| --- | --- | --- |
| Checksum mismatch | `401` | Use `mid|ENABLED_INSTRUMENTS|salt` |
| Merchant not enabled | `401` | Confirm merchant activation |
| Gateway error | `500` | Retry with backoff |
List all payment instruments enabled for your merchant account.

## Endpoint

```http
GET /vas/instruments
```

## Checksum Formula

```text
mid|ENABLED_INSTRUMENTS|salt
```

## Headers

| Header | Required | Description |
| --- | --- | --- |
| `X-Merchant-Id` | Yes | Merchant ID |
| `X-Checksum` | Yes | HMAC-SHA256 checksum |

## cURL

```bash
curl -X GET https://<uat-host>:8085/s2s/v1/vas/instruments \
  -H "X-Merchant-Id: B10001" \
  -H "X-Checksum: <checksum>"
```

## Success Response

```json
{
  "instruments": [
    { "paymentMode": "UPI", "paymentCode": "UPII", "name": "UPI Collect", "status": "ACTIVE" },
    { "paymentMode": "UPI", "paymentCode": "UPIC", "name": "UPI Intent", "status": "ACTIVE" },
    { "paymentMode": "UPI", "paymentCode": "DQR", "name": "Dynamic QR", "status": "ACTIVE" },
    { "paymentMode": "NB", "paymentCode": "HDFC", "name": "HDFC Bank Net Banking", "status": "ACTIVE" },
    { "paymentMode": "CC", "paymentCode": "CC", "name": "Credit Card", "status": "ACTIVE" },
    { "paymentMode": "DC", "paymentCode": "DC", "name": "Debit Card", "status": "ACTIVE" }
  ],
  "response": { "status": "SUCCESS", "statusCode": "200" }
}
```

## Response Fields

| Field | Type | Description |
| --- | --- | --- |
| `instruments[].paymentMode` | string | Payment mode to send in Initiate Transaction |
| `instruments[].paymentCode` | string | Payment code to send in Initiate Transaction |
| `instruments[].name` | string | Display name |
| `instruments[].status` | string | Instrument availability |

## Error Scenarios

| Scenario | HTTP Code | Resolution |
| --- | --- | --- |
| Checksum mismatch | `401` | Use `mid|ENABLED_INSTRUMENTS|salt` |
| Merchant not enabled | `401` | Confirm merchant activation |
| Gateway error | `500` | Retry with backoff |