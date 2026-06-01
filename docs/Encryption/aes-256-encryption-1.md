---
title: AES-256 Encryption
deprecated: false
hidden: false
metadata:
  robots: index
---
# AES-256 Encryption

## Overview

To ensure secure communication between your application and the Payment Gateway, all sensitive request and response payloads must be encrypted using AES-256-CBC encryption.

This document describes:

- Encryption algorithm
- Key generation
- Request encryption
- Response decryption
- Sample implementations
- Best practices

***

# Encryption Standard

| Parameter     | Value        |
| ------------- | ------------ |
| Algorithm     | AES          |
| Key Length    | 256 Bit      |
| Cipher Mode   | CBC          |
| Padding       | PKCS5Padding |
| Output Format | Base64       |
| Encoding      | UTF-8        |

***

# Encryption Flow

```text
Plain JSON
    ↓
AES-256-CBC Encryption
    ↓
Base64 Encoding
    ↓
Send to API
```

***

# Decryption Flow

```text
Encrypted Response
       ↓
Base64 Decode
       ↓
AES-256-CBC Decryption
       ↓
Plain JSON Response
```

***

# Request Structure

## Plain Request

```json
{
  "merchantId": "MID123456",
  "orderId": "ORD123456",
  "amount": "100.00",
  "currency": "INR"
}
```

***

## Encrypted Request

```json
{
  "data": "BASE64_ENCRYPTED_STRING"
}
```

***

# Response Structure

## Encrypted Response

```json
{
  "data": "BASE64_ENCRYPTED_STRING"
}
```

***

## Decrypted Response

```json
{
  "responseCode": "0",
  "description": "Success",
  "data": {
    "transactionId": "TXN123456789"
  }
}
```

***

# Encryption Parameters

## Secret Key

The merchant receives a unique:

```text
256-bit Secret Key
```

Example:

```text
12345678901234567890123456789012
```

***

## Initialization Vector (IV)

Example:

```text
1234567890123456
```

Length:

```text
16 Characters
```

***

# Java Encryption Example

```java
import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.util.Base64;

public class AESUtil {

    public static String encrypt(String data,
                                 String secretKey,
                                 String iv)
            throws Exception {

        Cipher cipher =
            Cipher.getInstance("AES/CBC/PKCS5Padding");

        SecretKeySpec keySpec =
            new SecretKeySpec(secretKey.getBytes(), "AES");

        IvParameterSpec ivSpec =
            new IvParameterSpec(iv.getBytes());

        cipher.init(Cipher.ENCRYPT_MODE, keySpec, ivSpec);

        byte[] encrypted =
            cipher.doFinal(data.getBytes());

        return Base64.getEncoder()
                .encodeToString(encrypted);
    }
}
```

***

# Java Decryption Example

```java
import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.util.Base64;

public class AESUtil {

    public static String decrypt(String encryptedData,
                                 String secretKey,
                                 String iv)
            throws Exception {

        Cipher cipher =
            Cipher.getInstance("AES/CBC/PKCS5Padding");

        SecretKeySpec keySpec =
            new SecretKeySpec(secretKey.getBytes(), "AES");

        IvParameterSpec ivSpec =
            new IvParameterSpec(iv.getBytes());

        cipher.init(Cipher.DECRYPT_MODE, keySpec, ivSpec);

        byte[] decrypted =
            cipher.doFinal(
                Base64.getDecoder()
                    .decode(encryptedData)
            );

        return new String(decrypted);
    }
}
```

***

# Node.js Encryption Example

```javascript
const crypto = require('crypto');

const encrypt = (
    text,
    key,
    iv
) => {

    const cipher =
        crypto.createCipheriv(
            'aes-256-cbc',
            Buffer.from(key),
            Buffer.from(iv)
        );

    let encrypted =
        cipher.update(
            text,
            'utf8',
            'base64'
        );

    encrypted += cipher.final('base64');

    return encrypted;
};
```

***

# Node.js Decryption Example

```javascript
const crypto = require('crypto');

const decrypt = (
    encryptedText,
    key,
    iv
) => {

    const decipher =
        crypto.createDecipheriv(
            'aes-256-cbc',
            Buffer.from(key),
            Buffer.from(iv)
        );

    let decrypted =
        decipher.update(
            encryptedText,
            'base64',
            'utf8'
        );

    decrypted += decipher.final('utf8');

    return decrypted;
};
```

***

# Security Recommendations

## DO

✅ Use HTTPS

✅ Store encryption keys securely

✅ Rotate keys periodically

✅ Restrict access to encryption keys

✅ Encrypt sensitive payloads

***

## DON'T

❌ Store keys in source code

❌ Expose keys in frontend applications

❌ Log decrypted customer information

❌ Share encryption credentials via email

***

# Common Errors

| Error               | Cause                     |
| ------------------- | ------------------------- |
| Invalid Key Length  | Incorrect AES Key         |
| Invalid IV Length   | IV must be 16 characters  |
| Padding Exception   | Wrong key or IV           |
| Base64 Decode Error | Invalid encrypted payload |
| Decryption Failed   | Corrupted data            |

***

# Troubleshooting

### Request Not Processing

Verify:

- Secret Key
- IV
- Encryption Algorithm
- Payload Format

***

### Response Cannot Be Decrypted

Verify:

- Same key used for encryption
- Same IV used for encryption
- Base64 string is intact

***

# Support

For integration assistance, provide:

- Merchant ID
- Request ID
- Timestamp
- Environment (UAT / Production)
- Error Screenshot
- Sample Payload

***

# Version History

| Version | Description     |
| ------- | --------------- |
| 1.0     | Initial Release |

<br />
