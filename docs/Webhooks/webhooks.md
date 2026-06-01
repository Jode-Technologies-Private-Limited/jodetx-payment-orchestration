---
title: Webhooks
deprecated: false
hidden: false
metadata:
  robots: index
---
**Here's the complete, clean, and professional Markdown file** ready for **readme.io**:

***

````markdown
# Payment Webhooks Documentation

Receive real-time notifications about payment events by integrating with our webhook system.

---

## Overview

Webhooks allow you to receive instant updates when a payment status changes. This is the most reliable way to confirm successful payments and handle failures.

Configure, verify, and process webhooks for the following events:
- `payment.success`
- `payment.failed`
- `payment.pending`

---

## Webhook Events

| Event               | Meaning                                           | Recommended Merchant Action                     |
|---------------------|---------------------------------------------------|-------------------------------------------------|
| `payment.success`   | Payment completed successfully                    | Mark order as paid, fulfill service, send confirmation |
| `payment.failed`    | Payment failed permanently                        | Mark order as failed, notify customer, allow retry |
| `payment.pending`   | Final status not yet determined                   | Keep order in pending state, wait for next update |

---

## Endpoint Requirements

Your webhook endpoint must meet the following requirements:

- **Protocol**: HTTPS only (publicly accessible)
- **Method**: `POST`
- **Content-Type**: `application/json`
- **Response**: Return HTTP `200 OK` within **5 seconds**
- **Signature Verification**: Mandatory
- **Idempotency**: Must handle duplicate events safely

### Example Endpoint URL

```http
POST https://yourdomain.com/webhooks/payment
````

***

## Webhook Payload

### Sample Payload

```json
{
  "event": "payment.success",
  "mid": "B10001",
  "orderNo": "ORD-20260529-0001",
  "rrn": "RRN123456789",
  "txnId": "TXN123456789",
  "txnAmount": "100.00",
  "paymentMode": "UPI",
  "paymentCode": "UPII",
  "status": "SUCCESS",
  "statusCode": "200",
  "message": "Payment successful",
  "eventTime": "2026-05-29T10:15:30+05:30"
}
```

### Field Reference

| Field         | Type   | Description                                   |
| ------------- | ------ | --------------------------------------------- |
| `event`       | string | Webhook event name (`payment.success`, etc.)  |
| `mid`         | string | Your Merchant ID                              |
| `orderNo`     | string | Your order number                             |
| `rrn`         | string | Retrieval Reference Number                    |
| `txnId`       | string | Gateway transaction ID                        |
| `txnAmount`   | string | Transaction amount (decimal string)           |
| `paymentMode` | string | Payment method (UPI, CARD, etc.)              |
| `paymentCode` | string | Payment code used                             |
| `status`      | string | Final status (`SUCCESS`, `FAILED`, `PENDING`) |
| `statusCode`  | string | Gateway status code                           |
| `message`     | string | Human readable message                        |
| `eventTime`   | string | ISO 8601 timestamp                            |

***

## Signature Verification

Every webhook request includes the header:

```http
X-Webhook-Signature: <hmac-sha256-signature>
```

### Node.js Example

```javascript
const crypto = require('crypto');

function verifyWebhook(rawBody, receivedSignature, secret) \{
  const expected = crypto
    .createHmac('sha256', secret)
    .update(rawBody, 'utf8')
    .digest('hex');

  return crypto.timingSafeEqual(
    Buffer.from(expected, 'hex'),
    Buffer.from(receivedSignature, 'hex')
  );
\}
```

> **Security Note**: Always verify the signature **before** processing any payload data.

***

## Retry Logic

If your server does not respond with `200 OK`, the system will retry delivery.

| Attempt | Delay      |
| ------- | ---------- |
| 1       | Immediate  |
| 2       | 5 minutes  |
| 3       | 15 minutes |
| 4       | 1 hour     |
| 5       | 6 hours    |

***

## Security Best Practices

- Always validate the `X-Webhook-Signature`
- Store raw webhook payloads for auditing
- Implement idempotency using `txnId` or `rrn`
- Do not rely on client-side redirects for payment confirmation
- Query transaction status API if webhook and your system are out of sync
- Use constant-time comparison for signature validation

***

## Implementation Tips

1. **Respond Quickly** — Return 200 immediately. Process business logic asynchronously.
2. **Idempotency** — Check if the transaction was already processed using `txnId`.
3. **Logging** — Log all incoming webhooks (raw body + headers).
4. **Error Handling** — Gracefully handle unexpected payloads.
5. **Testing** — Thoroughly test all three events in sandbox mode.

***

## Sample Code Structure (Express.js)

```javascript
const express = require('express');
const bodyParser = require('body-parser');
const crypto = require('crypto');

const app = express();
const WEBHOOK_SECRET = process.env.WEBHOOK_SECRET;

app.use(bodyParser.raw({ type: 'application/json' }));

app.post('/webhooks/payment', (req, res) => \{
  const signature = req.headers['x-webhook-signature'];
  const rawBody = req.body.toString();

  if (!verifyWebhook(rawBody, signature, WEBHOOK_SECRET)) {
    return res.status(401).send('Invalid signature');
  }

  const payload = JSON.parse(rawBody);

  // Process based on event
  switch (payload.event) {
    case 'payment.success':
      // Mark order as paid
      break;
    case 'payment.failed':
      // Mark order as failed
      break;
    case 'payment.pending':
      // Keep pending
      break;
  }

  res.status(200).send('OK');
\});
```

***

##

<br />
