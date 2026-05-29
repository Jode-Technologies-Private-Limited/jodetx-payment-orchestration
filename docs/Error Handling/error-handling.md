---
title: Error Handling
excerpt: >-
  HTTP codes, business statuses, retry rules, and failure handling guidance for
  S2S Payment Gateway APIs.
---
Handle S2S Payment Gateway errors by checking both the HTTP status code and the decrypted business response object.

## Error Response Model

Most responses are encrypted. After decrypting `encryptedData`, inspect the `response` object:

```json
{
  "response": {
    "status": "FAILED",
    "statusCode": "401",
    "message": "Invalid checksum"
  }
}
```

| Field | Type | Description |
| --- | --- | --- |
| `response.status` | string | Business status: `SUCCESS`, `FAILED`, or `PENDING` |
| `response.statusCode` | string | Gateway status code, usually aligned to HTTP code |
| `response.message` | string | Human-readable failure reason, when available |

<br />

## HTTP Status Codes

| HTTP Code | Meaning | Possible Causes | Resolution |
| --- | --- | --- | --- |
| `200 OK` | Request was accepted by the gateway | Payment may still be `SUCCESS`, `FAILED`, or `PENDING` at business level | Decrypt the response and evaluate `response.status` |
| `400 Bad Request` | Request payload is invalid | Missing required field, invalid JSON, unsupported `paymentMode`, malformed encrypted payload | Validate request schema and re-encrypt payload |
| `401 Unauthorized` | Authentication failed | Missing checksum, invalid checksum, inactive merchant ID, wrong environment salt | Regenerate checksum using the exact formula and verify credentials |
| `404 Not Found` | Endpoint or resource was not found | Wrong path, incorrect base URL, transaction ID not found | Confirm base URL, endpoint path, and transaction identifiers |
| `500 Internal Server Error` | Gateway or downstream bank/network issue | Temporary gateway error, bank timeout, switch failure | Retry using recommended backoff and query transaction status before reattempting payment |

<br />

## Business Status Values

| Status | Meaning | Merchant Action |
| --- | --- | --- |
| `SUCCESS` | Operation completed successfully | Update order/payment as successful |
| `FAILED` | Operation failed permanently | Show failure to customer and allow a new payment attempt |
| `PENDING` | Final payment state is not known yet | Poll Transaction Status or wait for webhook |

<Callout theme="warning" icon="fa-solid fa-triangle-exclamation">
Never mark an order as failed only because an API request timed out. Query Transaction Status first to avoid duplicate charges.
</Callout>

<br />

## Common Error Scenarios

### Invalid checksum

```json
{
  "response": {
    "status": "FAILED",
    "statusCode": "401",
    "message": "Invalid checksum"
  }
}
```

**Cause:** The checksum does not match the gateway-generated value.

**Resolution:** Rebuild the raw checksum string using the documented formula. Do not include extra fields, spaces, or formatted amount values.

### Invalid VPA

```json
{
  "vpaData": {
    "vpa": "invalid@upi",
    "isVPAValid": 0,
    "payerAccountName": ""
  },
  "response": {
    "status": "SUCCESS",
    "statusCode": "200"
  }
}
```

**Cause:** The Verify VPA API completed successfully, but the supplied VPA does not exist or cannot receive payments.

**Resolution:** Ask the customer to enter a valid UPI ID.

### Pending transaction

```json
{
  "txnId": "TXN123456789",
  "rrn": "RRN123456789",
  "response": {
    "status": "PENDING",
    "statusCode": "200",
    "message": "Transaction is pending at bank"
  }
}
```

**Cause:** The bank, UPI rail, or issuer has not returned a final status.

**Resolution:** Do not retry payment immediately. Poll Transaction Status or wait for webhook confirmation.

<br />

## Retry Policy

| Condition | Retry? | Recommended Action |
| --- | --- | --- |
| `400` validation error | No | Fix request payload |
| `401` checksum error | No | Fix authentication logic |
| `404` transaction not found | Conditional | Verify identifiers, then retry status query after 30 seconds |
| `500` gateway error | Yes | Retry with exponential backoff |
| Network timeout | Conditional | Query transaction status before creating a new transaction |
| `PENDING` business status | Yes, status only | Poll Transaction Status; do not initiate duplicate payment |

Recommended exponential backoff for retriable errors:

```text
Retry 1: after 5 seconds
Retry 2: after 15 seconds
Retry 3: after 30 seconds
Retry 4: after 60 seconds
Stop and reconcile manually after 4 failed attempts
```

<br />

## Error Handling Example

```javascript
async function handleGatewayResponse(httpStatus, decryptedBody) {
  if (httpStatus === 401) {
    throw new Error('Authentication failed. Verify checksum and merchant credentials.');
  }

  if (httpStatus >= 500) {
    return { action: 'RETRY_WITH_BACKOFF' };
  }

  const status = decryptedBody?.response?.status;

  if (status === 'SUCCESS') {
    return { action: 'MARK_SUCCESS', data: decryptedBody };
  }

  if (status === 'PENDING') {
    return { action: 'QUERY_STATUS_LATER', data: decryptedBody };
  }

  return { action: 'MARK_FAILED', data: decryptedBody };
}
```
Handle S2S Payment Gateway errors by checking both the HTTP status code and the decrypted business response object.

## Error Response Model

Most responses are encrypted. After decrypting `encryptedData`, inspect the `response` object:

```json
{
  "response": {
    "status": "FAILED",
    "statusCode": "401",
    "message": "Invalid checksum"
  }
}
```

| Field | Type | Description |
| --- | --- | --- |
| `response.status` | string | Business status: `SUCCESS`, `FAILED`, or `PENDING` |
| `response.statusCode` | string | Gateway status code, usually aligned to HTTP code |
| `response.message` | string | Human-readable failure reason, when available |

<br />

## HTTP Status Codes

| HTTP Code | Meaning | Possible Causes | Resolution |
| --- | --- | --- | --- |
| `200 OK` | Request was accepted by the gateway | Payment may still be `SUCCESS`, `FAILED`, or `PENDING` at business level | Decrypt the response and evaluate `response.status` |
| `400 Bad Request` | Request payload is invalid | Missing required field, invalid JSON, unsupported `paymentMode`, malformed encrypted payload | Validate request schema and re-encrypt payload |
| `401 Unauthorized` | Authentication failed | Missing checksum, invalid checksum, inactive merchant ID, wrong environment salt | Regenerate checksum using the exact formula and verify credentials |
| `404 Not Found` | Endpoint or resource was not found | Wrong path, incorrect base URL, transaction ID not found | Confirm base URL, endpoint path, and transaction identifiers |
| `500 Internal Server Error` | Gateway or downstream bank/network issue | Temporary gateway error, bank timeout, switch failure | Retry using recommended backoff and query transaction status before reattempting payment |

<br />

## Business Status Values

| Status | Meaning | Merchant Action |
| --- | --- | --- |
| `SUCCESS` | Operation completed successfully | Update order/payment as successful |
| `FAILED` | Operation failed permanently | Show failure to customer and allow a new payment attempt |
| `PENDING` | Final payment state is not known yet | Poll Transaction Status or wait for webhook |

<Callout theme="warning" icon="fa-solid fa-triangle-exclamation">
Never mark an order as failed only because an API request timed out. Query Transaction Status first to avoid duplicate charges.
</Callout>

<br />

## Common Error Scenarios

### Invalid checksum

```json
{
  "response": {
    "status": "FAILED",
    "statusCode": "401",
    "message": "Invalid checksum"
  }
}
```

**Cause:** The checksum does not match the gateway-generated value.

**Resolution:** Rebuild the raw checksum string using the documented formula. Do not include extra fields, spaces, or formatted amount values.

### Invalid VPA

```json
{
  "vpaData": {
    "vpa": "invalid@upi",
    "isVPAValid": 0,
    "payerAccountName": ""
  },
  "response": {
    "status": "SUCCESS",
    "statusCode": "200"
  }
}
```

**Cause:** The Verify VPA API completed successfully, but the supplied VPA does not exist or cannot receive payments.

**Resolution:** Ask the customer to enter a valid UPI ID.

### Pending transaction

```json
{
  "txnId": "TXN123456789",
  "rrn": "RRN123456789",
  "response": {
    "status": "PENDING",
    "statusCode": "200",
    "message": "Transaction is pending at bank"
  }
}
```

**Cause:** The bank, UPI rail, or issuer has not returned a final status.

**Resolution:** Do not retry payment immediately. Poll Transaction Status or wait for webhook confirmation.

<br />

## Retry Policy

| Condition | Retry? | Recommended Action |
| --- | --- | --- |
| `400` validation error | No | Fix request payload |
| `401` checksum error | No | Fix authentication logic |
| `404` transaction not found | Conditional | Verify identifiers, then retry status query after 30 seconds |
| `500` gateway error | Yes | Retry with exponential backoff |
| Network timeout | Conditional | Query transaction status before creating a new transaction |
| `PENDING` business status | Yes, status only | Poll Transaction Status; do not initiate duplicate payment |

Recommended exponential backoff for retriable errors:

```text
Retry 1: after 5 seconds
Retry 2: after 15 seconds
Retry 3: after 30 seconds
Retry 4: after 60 seconds
Stop and reconcile manually after 4 failed attempts
```

<br />

## Error Handling Example

```javascript
async function handleGatewayResponse(httpStatus, decryptedBody) {
  if (httpStatus === 401) {
    throw new Error('Authentication failed. Verify checksum and merchant credentials.');
  }

  if (httpStatus >= 500) {
    return { action: 'RETRY_WITH_BACKOFF' };
  }

  const status = decryptedBody?.response?.status;

  if (status === 'SUCCESS') {
    return { action: 'MARK_SUCCESS', data: decryptedBody };
  }

  if (status === 'PENDING') {
    return { action: 'QUERY_STATUS_LATER', data: decryptedBody };
  }

  return { action: 'MARK_FAILED', data: decryptedBody };
}
```
Handle S2S Payment Gateway errors by checking both the HTTP status code and the decrypted business response object.

## Error Response Model

Most responses are encrypted. After decrypting `encryptedData`, inspect the `response` object:

```json
{
  "response": {
    "status": "FAILED",
    "statusCode": "401",
    "message": "Invalid checksum"
  }
}
```

| Field | Type | Description |
| --- | --- | --- |
| `response.status` | string | Business status: `SUCCESS`, `FAILED`, or `PENDING` |
| `response.statusCode` | string | Gateway status code, usually aligned to HTTP code |
| `response.message` | string | Human-readable failure reason, when available |

<br />

## HTTP Status Codes

| HTTP Code | Meaning | Possible Causes | Resolution |
| --- | --- | --- | --- |
| `200 OK` | Request was accepted by the gateway | Payment may still be `SUCCESS`, `FAILED`, or `PENDING` at business level | Decrypt the response and evaluate `response.status` |
| `400 Bad Request` | Request payload is invalid | Missing required field, invalid JSON, unsupported `paymentMode`, malformed encrypted payload | Validate request schema and re-encrypt payload |
| `401 Unauthorized` | Authentication failed | Missing checksum, invalid checksum, inactive merchant ID, wrong environment salt | Regenerate checksum using the exact formula and verify credentials |
| `404 Not Found` | Endpoint or resource was not found | Wrong path, incorrect base URL, transaction ID not found | Confirm base URL, endpoint path, and transaction identifiers |
| `500 Internal Server Error` | Gateway or downstream bank/network issue | Temporary gateway error, bank timeout, switch failure | Retry using recommended backoff and query transaction status before reattempting payment |

<br />

## Business Status Values

| Status | Meaning | Merchant Action |
| --- | --- | --- |
| `SUCCESS` | Operation completed successfully | Update order/payment as successful |
| `FAILED` | Operation failed permanently | Show failure to customer and allow a new payment attempt |
| `PENDING` | Final payment state is not known yet | Poll Transaction Status or wait for webhook |

<Callout theme="warning" icon="fa-solid fa-triangle-exclamation">
Never mark an order as failed only because an API request timed out. Query Transaction Status first to avoid duplicate charges.
</Callout>

<br />

## Common Error Scenarios

### Invalid checksum

```json
{
  "response": {
    "status": "FAILED",
    "statusCode": "401",
    "message": "Invalid checksum"
  }
}
```

**Cause:** The checksum does not match the gateway-generated value.

**Resolution:** Rebuild the raw checksum string using the documented formula. Do not include extra fields, spaces, or formatted amount values.

### Invalid VPA

```json
{
  "vpaData": {
    "vpa": "invalid@upi",
    "isVPAValid": 0,
    "payerAccountName": ""
  },
  "response": {
    "status": "SUCCESS",
    "statusCode": "200"
  }
}
```

**Cause:** The Verify VPA API completed successfully, but the supplied VPA does not exist or cannot receive payments.

**Resolution:** Ask the customer to enter a valid UPI ID.

### Pending transaction

```json
{
  "txnId": "TXN123456789",
  "rrn": "RRN123456789",
  "response": {
    "status": "PENDING",
    "statusCode": "200",
    "message": "Transaction is pending at bank"
  }
}
```

**Cause:** The bank, UPI rail, or issuer has not returned a final status.

**Resolution:** Do not retry payment immediately. Poll Transaction Status or wait for webhook confirmation.

<br />

## Retry Policy

| Condition | Retry? | Recommended Action |
| --- | --- | --- |
| `400` validation error | No | Fix request payload |
| `401` checksum error | No | Fix authentication logic |
| `404` transaction not found | Conditional | Verify identifiers, then retry status query after 30 seconds |
| `500` gateway error | Yes | Retry with exponential backoff |
| Network timeout | Conditional | Query transaction status before creating a new transaction |
| `PENDING` business status | Yes, status only | Poll Transaction Status; do not initiate duplicate payment |

Recommended exponential backoff for retriable errors:

```text
Retry 1: after 5 seconds
Retry 2: after 15 seconds
Retry 3: after 30 seconds
Retry 4: after 60 seconds
Stop and reconcile manually after 4 failed attempts
```

<br />

## Error Handling Example

```javascript
async function handleGatewayResponse(httpStatus, decryptedBody) {
  if (httpStatus === 401) {
    throw new Error('Authentication failed. Verify checksum and merchant credentials.');
  }

  if (httpStatus >= 500) {
    return { action: 'RETRY_WITH_BACKOFF' };
  }

  const status = decryptedBody?.response?.status;

  if (status === 'SUCCESS') {
    return { action: 'MARK_SUCCESS', data: decryptedBody };
  }

  if (status === 'PENDING') {
    return { action: 'QUERY_STATUS_LATER', data: decryptedBody };
  }

  return { action: 'MARK_FAILED', data: decryptedBody };
}
```