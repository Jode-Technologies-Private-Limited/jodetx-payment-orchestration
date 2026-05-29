---
title: AES-256 Encryption
excerpt: >-
  Encrypt API requests and decrypt responses using AES-256-CBC for the S2S
  Payment Gateway.
---
Encrypt every S2S Payment Gateway request and decrypt every response using AES-256-CBC with your merchant-specific encryption key and IV.

## Encryption Model

The gateway uses symmetric encryption: the same AES-256 key and initialization vector (IV) are used by your server and the gateway to encrypt and decrypt payloads.

```
Merchant Server                    S2S Payment Gateway
      │                                     │
      │ 1. Build JSON payload               │
      │ 2. Encrypt with AES-256-CBC         │
      │ 3. Send encryptedData over HTTPS    │
      ├────────────────────────────────────►│
      │                                     │ 4. Decrypt payload
      │                                     │ 5. Process request
      │                                     │ 6. Encrypt response
      │ 7. Receive encryptedData            │
      │◄────────────────────────────────────┤
      │ 8. Decrypt and parse JSON           │
```

<br />

## Encrypted Request Format

All JSON request bodies are wrapped as Base64-encoded ciphertext:

```json
{
  "encryptedData": "<base64-aes-256-cbc-ciphertext>"
}
```

The plaintext before encryption is the normal API-specific JSON payload.

Example plaintext for Verify VPA:

```json
{
  "mid": "B10001",
  "vpa": "8898327678@upi"
}
```

<br />

## Encrypted Response Format

Successful and error responses use the same wrapper:

```json
{
  "encryptedData": "<base64-aes-256-cbc-ciphertext>"
}
```

After decryption, the response is a regular JSON object:

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

<br />

## Algorithm Requirements

| Property | Value |
| --- | --- |
| Algorithm | AES |
| Key size | 256 bits |
| Mode | CBC |
| Padding | PKCS#5 / PKCS#7 |
| Ciphertext encoding | Base64 |
| Key encoding | Hexadecimal or raw bytes, as provided during onboarding |

<Callout theme="warning" icon="fa-solid fa-triangle-exclamation">
Use the exact key and IV format provided during onboarding. If your credentials are supplied as hex strings, decode them before creating the cipher.
</Callout>

<br />

## Code Samples

<Tabs>
<Tab title="Java">
```java
import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;
import java.util.Base64;

public final class Aes256Cbc {
    private Aes256Cbc() {}

    public static String encrypt(String plainText, byte[] key, byte[] iv) throws Exception {
        Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
        cipher.init(Cipher.ENCRYPT_MODE, new SecretKeySpec(key, "AES"), new IvParameterSpec(iv));
        byte[] encrypted = cipher.doFinal(plainText.getBytes(StandardCharsets.UTF_8));
        return Base64.getEncoder().encodeToString(encrypted);
    }

    public static String decrypt(String encryptedBase64, byte[] key, byte[] iv) throws Exception {
        Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
        cipher.init(Cipher.DECRYPT_MODE, new SecretKeySpec(key, "AES"), new IvParameterSpec(iv));
        byte[] decoded = Base64.getDecoder().decode(encryptedBase64);
        return new String(cipher.doFinal(decoded), StandardCharsets.UTF_8);
    }
}
```
</Tab>
<Tab title="Node.js">
```javascript
const crypto = require('crypto');

function encryptJson(payload, keyHex, ivHex) {
  const cipher = crypto.createCipheriv(
    'aes-256-cbc',
    Buffer.from(keyHex, 'hex'),
    Buffer.from(ivHex, 'hex')
  );

  let encrypted = cipher.update(JSON.stringify(payload), 'utf8', 'base64');
  encrypted += cipher.final('base64');
  return encrypted;
}

function decryptJson(encryptedBase64, keyHex, ivHex) {
  const decipher = crypto.createDecipheriv(
    'aes-256-cbc',
    Buffer.from(keyHex, 'hex'),
    Buffer.from(ivHex, 'hex')
  );

  let decrypted = decipher.update(encryptedBase64, 'base64', 'utf8');
  decrypted += decipher.final('utf8');
  return JSON.parse(decrypted);
}

module.exports = { encryptJson, decryptJson };
```
</Tab>
<Tab title="PHP">
```php
<?php
function encryptJson(array $payload, string $keyHex, string $ivHex): string {
    $key = hex2bin($keyHex);
    $iv  = hex2bin($ivHex);
    $plaintext = json_encode($payload, JSON_UNESCAPED_SLASHES);

    return openssl_encrypt(
        $plaintext,
        'AES-256-CBC',
        $key,
        OPENSSL_RAW_DATA,
        $iv
    );
}

function decryptJson(string $encryptedBase64, string $keyHex, string $ivHex): array {
    $key = hex2bin($keyHex);
    $iv  = hex2bin($ivHex);
    $ciphertext = base64_decode($encryptedBase64);

    $plaintext = openssl_decrypt(
        $ciphertext,
        'AES-256-CBC',
        $key,
        OPENSSL_RAW_DATA,
        $iv
    );

    return json_decode($plaintext, true);
}
?>
```
</Tab>
<Tab title="Python">
```python
import base64
import binascii
import json
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad

def encrypt_json(payload: dict, key_hex: str, iv_hex: str) -> str:
    key = binascii.unhexlify(key_hex)
    iv = binascii.unhexlify(iv_hex)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    plaintext = json.dumps(payload, separators=(",", ":")).encode()
    ciphertext = cipher.encrypt(pad(plaintext, AES.block_size))
    return base64.b64encode(ciphertext).decode()

def decrypt_json(encrypted_base64: str, key_hex: str, iv_hex: str) -> dict:
    key = binascii.unhexlify(key_hex)
    iv = binascii.unhexlify(iv_hex)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    plaintext = unpad(cipher.decrypt(base64.b64decode(encrypted_base64)), AES.block_size)
    return json.loads(plaintext)
```
</Tab>
</Tabs>

<br />

## Security Best Practices

1. Store encryption keys in a managed secret store.
2. Rotate keys only through a coordinated gateway-supported process.
3. Never log plaintext request or response bodies in production.
4. Mask PAN, CVV, VPA, phone number, and email fields in logs.
5. Reject decrypted responses that cannot be parsed as valid JSON.
6. Enforce TLS 1.2 or higher for outbound requests.
7. Apply PCI-DSS controls when handling card data server-side.

<br />

## Common Encryption Errors

| Error | Cause | Resolution |
| --- | --- | --- |
| `Invalid key length` | Key was not decoded correctly | Confirm whether key is hex, Base64, or raw text |
| `bad decrypt` | Wrong key, IV, or padding | Validate environment credentials and AES mode |
| Empty decrypted payload | Response body was not Base64 decoded | Base64-decode before decrypting |
| JSON parse failure | Decrypted text is not valid JSON | Log sanitized ciphertext reference ID and contact support |
Encrypt every S2S Payment Gateway request and decrypt every response using AES-256-CBC with your merchant-specific encryption key and IV.

## Encryption Model

The gateway uses symmetric encryption: the same AES-256 key and initialization vector (IV) are used by your server and the gateway to encrypt and decrypt payloads.

```
Merchant Server                    S2S Payment Gateway
      │                                     │
      │ 1. Build JSON payload               │
      │ 2. Encrypt with AES-256-CBC         │
      │ 3. Send encryptedData over HTTPS    │
      ├────────────────────────────────────►│
      │                                     │ 4. Decrypt payload
      │                                     │ 5. Process request
      │                                     │ 6. Encrypt response
      │ 7. Receive encryptedData            │
      │◄────────────────────────────────────┤
      │ 8. Decrypt and parse JSON           │
```

<br />

## Encrypted Request Format

All JSON request bodies are wrapped as Base64-encoded ciphertext:

```json
{
  "encryptedData": "<base64-aes-256-cbc-ciphertext>"
}
```

The plaintext before encryption is the normal API-specific JSON payload.

Example plaintext for Verify VPA:

```json
{
  "mid": "B10001",
  "vpa": "8898327678@upi"
}
```

<br />

## Encrypted Response Format

Successful and error responses use the same wrapper:

```json
{
  "encryptedData": "<base64-aes-256-cbc-ciphertext>"
}
```

After decryption, the response is a regular JSON object:

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

<br />

## Algorithm Requirements

| Property | Value |
| --- | --- |
| Algorithm | AES |
| Key size | 256 bits |
| Mode | CBC |
| Padding | PKCS#5 / PKCS#7 |
| Ciphertext encoding | Base64 |
| Key encoding | Hexadecimal or raw bytes, as provided during onboarding |

<Callout theme="warning" icon="fa-solid fa-triangle-exclamation">
Use the exact key and IV format provided during onboarding. If your credentials are supplied as hex strings, decode them before creating the cipher.
</Callout>

<br />

## Code Samples

<Tabs>
<Tab title="Java">
```java
import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;
import java.util.Base64;

public final class Aes256Cbc {
    private Aes256Cbc() {}

    public static String encrypt(String plainText, byte[] key, byte[] iv) throws Exception {
        Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
        cipher.init(Cipher.ENCRYPT_MODE, new SecretKeySpec(key, "AES"), new IvParameterSpec(iv));
        byte[] encrypted = cipher.doFinal(plainText.getBytes(StandardCharsets.UTF_8));
        return Base64.getEncoder().encodeToString(encrypted);
    }

    public static String decrypt(String encryptedBase64, byte[] key, byte[] iv) throws Exception {
        Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
        cipher.init(Cipher.DECRYPT_MODE, new SecretKeySpec(key, "AES"), new IvParameterSpec(iv));
        byte[] decoded = Base64.getDecoder().decode(encryptedBase64);
        return new String(cipher.doFinal(decoded), StandardCharsets.UTF_8);
    }
}
```
</Tab>
<Tab title="Node.js">
```javascript
const crypto = require('crypto');

function encryptJson(payload, keyHex, ivHex) {
  const cipher = crypto.createCipheriv(
    'aes-256-cbc',
    Buffer.from(keyHex, 'hex'),
    Buffer.from(ivHex, 'hex')
  );

  let encrypted = cipher.update(JSON.stringify(payload), 'utf8', 'base64');
  encrypted += cipher.final('base64');
  return encrypted;
}

function decryptJson(encryptedBase64, keyHex, ivHex) {
  const decipher = crypto.createDecipheriv(
    'aes-256-cbc',
    Buffer.from(keyHex, 'hex'),
    Buffer.from(ivHex, 'hex')
  );

  let decrypted = decipher.update(encryptedBase64, 'base64', 'utf8');
  decrypted += decipher.final('utf8');
  return JSON.parse(decrypted);
}

module.exports = { encryptJson, decryptJson };
```
</Tab>
<Tab title="PHP">
```php
<?php
function encryptJson(array $payload, string $keyHex, string $ivHex): string {
    $key = hex2bin($keyHex);
    $iv  = hex2bin($ivHex);
    $plaintext = json_encode($payload, JSON_UNESCAPED_SLASHES);

    return openssl_encrypt(
        $plaintext,
        'AES-256-CBC',
        $key,
        OPENSSL_RAW_DATA,
        $iv
    );
}

function decryptJson(string $encryptedBase64, string $keyHex, string $ivHex): array {
    $key = hex2bin($keyHex);
    $iv  = hex2bin($ivHex);
    $ciphertext = base64_decode($encryptedBase64);

    $plaintext = openssl_decrypt(
        $ciphertext,
        'AES-256-CBC',
        $key,
        OPENSSL_RAW_DATA,
        $iv
    );

    return json_decode($plaintext, true);
}
?>
```
</Tab>
<Tab title="Python">
```python
import base64
import binascii
import json
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad

def encrypt_json(payload: dict, key_hex: str, iv_hex: str) -> str:
    key = binascii.unhexlify(key_hex)
    iv = binascii.unhexlify(iv_hex)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    plaintext = json.dumps(payload, separators=(",", ":")).encode()
    ciphertext = cipher.encrypt(pad(plaintext, AES.block_size))
    return base64.b64encode(ciphertext).decode()

def decrypt_json(encrypted_base64: str, key_hex: str, iv_hex: str) -> dict:
    key = binascii.unhexlify(key_hex)
    iv = binascii.unhexlify(iv_hex)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    plaintext = unpad(cipher.decrypt(base64.b64decode(encrypted_base64)), AES.block_size)
    return json.loads(plaintext)
```
</Tab>
</Tabs>

<br />

## Security Best Practices

1. Store encryption keys in a managed secret store.
2. Rotate keys only through a coordinated gateway-supported process.
3. Never log plaintext request or response bodies in production.
4. Mask PAN, CVV, VPA, phone number, and email fields in logs.
5. Reject decrypted responses that cannot be parsed as valid JSON.
6. Enforce TLS 1.2 or higher for outbound requests.
7. Apply PCI-DSS controls when handling card data server-side.

<br />

## Common Encryption Errors

| Error | Cause | Resolution |
| --- | --- | --- |
| `Invalid key length` | Key was not decoded correctly | Confirm whether key is hex, Base64, or raw text |
| `bad decrypt` | Wrong key, IV, or padding | Validate environment credentials and AES mode |
| Empty decrypted payload | Response body was not Base64 decoded | Base64-decode before decrypting |
| JSON parse failure | Decrypted text is not valid JSON | Log sanitized ciphertext reference ID and contact support |