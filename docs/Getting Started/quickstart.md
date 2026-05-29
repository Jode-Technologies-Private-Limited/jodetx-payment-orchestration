---
title: Quick Start
excerpt: Get your first S2S Payment Gateway transaction running in under 10 minutes.
---
Make your first successful test transaction in under 10 minutes by following these five steps.

## Prerequisites

Before you start, complete the following:

1. **Request sandbox credentials** — Contact your account manager to receive your UAT `mid`, `salt`, encryption key, and IV.
2. **Set your base URL** — All UAT requests go to `https://<uat-host>:8085/s2s/v1`.
3. **Install a REST client** — Use cURL, Postman, or any HTTP client that supports custom headers.

<br />

## Step 1 — Generate a Checksum

Every API request requires an `X-Checksum` header (or includes a checksum field). The checksum is an HMAC-SHA256 hash of a pipe-delimited string of request parameters, with your `salt` appended last.

For Verify VPA, the formula is: `mid|VPA_VERIFY|vpa|salt`

<Tabs>
<Tab title="Node.js">
```javascript
const crypto = require('crypto');

const mid    = 'B10001';
const vpa    = '8898327678@upi';
const salt   = 'your-secret-salt';  // Keep this server-side only

const raw    = `${mid}|VPA_VERIFY|${vpa}|${salt}`;
const checksum = crypto.createHmac('sha256', salt).update(raw).digest('hex');

console.log(checksum);
// e.g. "3d4f2a1b9c8e7f6a5b4c3d2e1f0a9b8c7d6e5f4a3b2c1d0e9f8a7b6c5d4e3f2"
```
</Tab>
<Tab title="Python">
```python
import hmac
import hashlib

mid    = 'B10001'
vpa    = '8898327678@upi'
salt   = 'your-secret-salt'  # Keep this server-side only

raw    = f"{mid}|VPA_VERIFY|{vpa}|{salt}"
checksum = hmac.new(salt.encode(), raw.encode(), hashlib.sha256).hexdigest()

print(checksum)
```
</Tab>
<Tab title="Java">
```java
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.util.HexFormat;

public class ChecksumUtil {
    public static String generate(String mid, String vpa, String salt) throws Exception {
        String raw = mid + "|VPA_VERIFY|" + vpa + "|" + salt;
        Mac mac = Mac.getInstance("HmacSHA256");
        mac.init(new SecretKeySpec(salt.getBytes("UTF-8"), "HmacSHA256"));
        return HexFormat.of().formatHex(mac.doFinal(raw.getBytes("UTF-8")));
    }
}
```
</Tab>
<Tab title="PHP">
```php
<?php
$mid  = 'B10001';
$vpa  = '8898327678@upi';
$salt = 'your-secret-salt'; // Keep this server-side only

$raw      = "{$mid}|VPA_VERIFY|{$vpa}|{$salt}";
$checksum = hash_hmac('sha256', $raw, $salt);

echo $checksum;
?>
```
</Tab>
</Tabs>

<br />

## Step 2 — Encrypt the Request Body

Request bodies are encrypted with AES-256-CBC before being sent. See the [Encryption Guide](/docs/encryption) for the full implementation. For this quick start, use the helper snippets below.

<Tabs>
<Tab title="Node.js">
```javascript
const crypto = require('crypto');

function encryptPayload(data, key, iv) {
  // key and iv are hex strings provided by the gateway
  const keyBuf  = Buffer.from(key, 'hex');
  const ivBuf   = Buffer.from(iv, 'hex');
  const cipher  = crypto.createCipheriv('aes-256-cbc', keyBuf, ivBuf);
  let encrypted = cipher.update(JSON.stringify(data), 'utf8', 'base64');
  encrypted    += cipher.final('base64');
  return encrypted;
}

const payload   = { mid: 'B10001', vpa: '8898327678@upi' };
const encrypted = encryptPayload(payload, process.env.ENC_KEY, process.env.ENC_IV);
```
</Tab>
<Tab title="Python">
```python
import json, base64
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
import binascii

def encrypt_payload(data: dict, key_hex: str, iv_hex: str) -> str:
    key    = binascii.unhexlify(key_hex)
    iv     = binascii.unhexlify(iv_hex)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    ct     = cipher.encrypt(pad(json.dumps(data).encode(), AES.block_size))
    return base64.b64encode(ct).decode()

payload   = {"mid": "B10001", "vpa": "8898327678@upi"}
encrypted = encrypt_payload(payload, ENC_KEY, ENC_IV)
```
</Tab>
</Tabs>

<br />

## Step 3 — Send the API Request

Use the encrypted payload and checksum header to call Verify VPA:

```bash title="cURL — Verify VPA"
curl -X POST https://<uat-host>:8085/s2s/v1/vas/verify_vpa \
  -H "Content-Type: application/json" \
  -H "X-Merchant-Id: B10001" \
  -H "X-Checksum: <generated-checksum>" \
  -d '{"encryptedData":"<base64-encrypted-payload>"}'
```

<br />

## Step 4 — Decrypt the Response

Responses are returned as AES-256-CBC encrypted Base64 strings. Decrypt them with the same key and IV:

<Tabs>
<Tab title="Node.js">
```javascript
function decryptResponse(encryptedBase64, key, iv) {
  const keyBuf    = Buffer.from(key, 'hex');
  const ivBuf     = Buffer.from(iv, 'hex');
  const decipher  = crypto.createDecipheriv('aes-256-cbc', keyBuf, ivBuf);
  let decrypted   = decipher.update(encryptedBase64, 'base64', 'utf8');
  decrypted      += decipher.final('utf8');
  return JSON.parse(decrypted);
}

const result = decryptResponse(response.encryptedData, process.env.ENC_KEY, process.env.ENC_IV);
console.log(result);
// {
//   "vpaData": { "vpa": "8898327678@upi", "isVPAValid": 1, "payerAccountName": "" },
//   "response": { "status": "SUCCESS", "statusCode": "200" }
// }
```
</Tab>
<Tab title="Python">
```python
from Crypto.Util.Padding import unpad

def decrypt_response(encrypted_b64: str, key_hex: str, iv_hex: str) -> dict:
    key       = binascii.unhexlify(key_hex)
    iv        = binascii.unhexlify(iv_hex)
    cipher    = AES.new(key, AES.MODE_CBC, iv)
    plaintext = unpad(cipher.decrypt(base64.b64decode(encrypted_b64)), AES.block_size)
    return json.loads(plaintext)

result = decrypt_response(response_json["encryptedData"], ENC_KEY, ENC_IV)
print(result)
```
</Tab>
</Tabs>

<br />

## Step 5 — Handle the Response

Check `response.status` and `response.statusCode` to determine the outcome:

```javascript title="Response handling"
if (result.response.status === 'SUCCESS' && result.vpaData.isVPAValid === 1) {
  // VPA is valid — proceed to initiate UPI Collect transaction
  console.log('Valid VPA:', result.vpaData.vpa);
} else if (result.vpaData.isVPAValid === 0) {
  // VPA not found — prompt customer to re-enter UPI ID
  console.error('Invalid VPA');
} else {
  // Gateway error — log statusCode and retry
  console.error('Gateway error:', result.response.statusCode);
}
```

<br />

## What's Next

<Cards columns={2}>
  <Card title="Initiate a UPI Transaction" href="/reference/initiate-transaction" icon="fa-duotone fa-money-check-dollar">
    After verifying the VPA, initiate a UPI Collect payment.
  </Card>
  <Card title="Check Transaction Status" href="/reference/transaction-status" icon="fa-duotone fa-list-check">
    Poll or receive webhook notifications for payment status.
  </Card>
  <Card title="Authentication Deep Dive" href="/docs/authentication" icon="fa-duotone fa-key">
    Understand checksum generation for all seven API endpoints.
  </Card>
  <Card title="Go-Live Checklist" href="/docs/production-checklist" icon="fa-duotone fa-clipboard-check">
    Review everything before switching to production credentials.
  </Card>
</Cards>
