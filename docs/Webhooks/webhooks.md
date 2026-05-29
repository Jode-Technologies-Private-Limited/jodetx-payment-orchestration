---
title: Webhooks
excerpt: >-
  Configure, verify, and process payment webhooks for success, failed, and
  pending transaction events.
---
Receive real-time payment status updates by exposing a secure HTTPS webhook endpoint on your server.

## Webhook Events

| Event | Meaning | Merchant Action |
| --- | --- | --- |
| `payment.success` | Payment completed successfully | Mark order as paid and fulfill service |
| `payment.failed` | Payment failed permanently | Mark order as failed and allow retry |
| `payment.pending` | Payment final state is not yet available | Keep order pending and wait for next update |

<br />

## Webhook Endpoint Requirements

1. Expose an HTTPS endpoint reachable from the public internet.
2. Accept `POST` requests with `Content-Type: application/json`.
3. Verify the webhook signature before processing the event.
4. Return HTTP `200` within 5 seconds after accepting the event.
5. Process duplicate events idempotently using `txnId`, `rrn`, or event ID.

Example endpoint:

```text
POST https://merchant.example.com/webhooks/payment-gateway
```

<br />

## Webhook Payload

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

| Field | Type | Description |
| --- | --- | --- |
| `event` | string | Webhook event name |
| `mid` | string | Merchant ID |
| `orderNo` | string | Merchant order number |
| `rrn` | string | Retrieval reference number |
| `txnId` | string | Gateway transaction ID |
| `txnAmount` | string | Transaction amount as a decimal string |
| `paymentMode` | string | Payment mode used by the customer |
| `paymentCode` | string | Payment code used for the transaction |
| `status` | string | Final or current transaction status |
| `statusCode` | string | Gateway status code |
| `message` | string | Human-readable status message |
| `eventTime` | string | ISO-8601 timestamp of the event |

<br />

## Signature Validation

Webhook requests include a signature header:

```text
X-Webhook-Signature: <hmac-sha256-signature>
```

Generate the expected signature from the raw request body using your webhook secret or merchant salt, then compare it with `X-Webhook-Signature` using a constant-time comparison.

<Tabs>
<Tab title="Node.js">
```javascript
const crypto = require('crypto');

function verifyWebhook(rawBody, receivedSignature, secret) {
  const expected = crypto
    .createHmac('sha256', secret)
    .update(rawBody, 'utf8')
    .digest('hex');

  return crypto.timingSafeEqual(
    Buffer.from(expected, 'hex'),
    Buffer.from(receivedSignature, 'hex')
  );
}
```
</Tab>
<Tab title="Python">
```python
import hmac
import hashlib

def verify_webhook(raw_body: bytes, received_signature: str, secret: str) -> bool:
    expected = hmac.new(secret.encode(), raw_body, hashlib.sha256).hexdigest()
    return hmac.compare_digest(expected, received_signature)
```
</Tab>
</Tabs>

<br />

## Retry Logic

If your endpoint does not return HTTP `200`, the gateway retries delivery.

Recommended retry schedule:

| Attempt | Delay |
| --- | --- |
| 1 | Immediate |
| 2 | 5 minutes |
| 3 | 15 minutes |
| 4 | 1 hour |
| 5 | 6 hours |

Your endpoint must handle duplicates because the same event may be delivered more than once.

<br />

## Security Checklist

- Verify `X-Webhook-Signature` before reading business fields.
- Store raw webhook payloads for audit and dispute investigations.
- Reject requests older than your accepted replay window if a timestamp header is configured.
- Do not trust client-side redirect status as payment confirmation.
- Query Transaction Status when webhook and internal order status conflict.
