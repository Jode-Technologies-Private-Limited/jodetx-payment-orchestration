---
title: HMAC-SHA256 Authentication
excerpt: >-
  Generate and validate HMAC-SHA256 checksums for S2S Payment Gateway API
  requests.
---
Authenticate every S2S Payment Gateway request by generating and sending an HMAC-SHA256 checksum built from endpoint-specific request parameters.

## How Authentication Works

Each merchant receives a unique:

| Credential | Description | Storage Requirement |
| --- | --- | --- |
| `mid` | Merchant identifier assigned during onboarding | Server-side configuration |
| `salt` | Secret value used to generate HMAC-SHA256 checksums | Secret manager or encrypted environment variable |
| Encryption key | AES-256 key for request and response encryption | Secret manager only |
| IV | Initialization vector for AES-256-CBC | Secret manager only |

For every API request, your server:

1. Builds the raw checksum string using the API-specific formula.
2. Appends `salt` as the final component of the formula.
3. Generates an HMAC-SHA256 digest using `salt` as the signing key.
4. Sends the checksum in the `X-Checksum` header.

<Callout theme="danger" icon="fa-solid fa-shield-xmark">
Never generate checksums in browser JavaScript or mobile apps. The `salt` must remain only on your backend.
</Callout>

<br />

## Checksum Header

| Header | Required | Description |
| --- | --- | --- |
| `X-Merchant-Id` | Yes | Your merchant ID, for example `B10001` |
| `X-Checksum` | Yes | HMAC-SHA256 checksum generated from the endpoint formula |
| `Content-Type` | Yes for POST | Must be `application/json` |

<br />

## Checksum Formulas

| API | Method | Formula |
| --- | --- | --- |
| Verify VPA | `POST /vas/verify_vpa` | `mid\|VPA_VERIFY\|vpa\|salt` |
| Netbanking List | `GET /vas/netbanking` | `mid\|NETBANKING_LIST\|salt` |
| Enabled Instruments | `GET /vas/instruments` | `mid\|ENABLED_INSTRUMENTS\|salt` |
| Initiate Transaction | `POST /payment/initiate` | `mid\|orderNo\|rrn\|txnAmount\|paymentMode\|paymentCode\|salt` |
| Transaction Status | `POST /merchant/v1/getTxnStatus` | `mid\|rrn\|txnId\|salt` |

<br />

## Validation Rules

The gateway validates checksums using the same formula and merchant salt. A request is rejected when:

- The `X-Checksum` header is missing.
- The `X-Merchant-Id` header is invalid or inactive.
- A required field used in the formula is missing.
- Field ordering in the raw string is incorrect.
- Extra whitespace or case changes are introduced before hashing.
- The request was signed with the wrong environment's salt.

<br />

## Checksum Example — Verify VPA

Request payload:

```json
{
  "mid": "B10001",
  "vpa": "8898327678@upi"
}
```

Raw checksum string:

```text
B10001|VPA_VERIFY|8898327678@upi|your-secret-salt
```

Generated checksum:

```text
3d4f2a1b9c8e7f6a5b4c3d2e1f0a9b8c7d6e5f4a3b2c1d0e9f8a7b6c5d4e3f2
```

HTTP request:

```bash
curl -X POST https://<uat-host>:8085/s2s/v1/vas/verify_vpa \
  -H "Content-Type: application/json" \
  -H "X-Merchant-Id: B10001" \
  -H "X-Checksum: 3d4f2a1b9c8e7f6a5b4c3d2e1f0a9b8c7d6e5f4a3b2c1d0e9f8a7b6c5d4e3f2" \
  -d '{"encryptedData":"<encrypted-payload>"}'
```

<br />

## Code Samples

<Tabs>
<Tab title="Java">
```java
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;
import java.util.HexFormat;

public final class ChecksumGenerator {
    private ChecksumGenerator() {}

    public static String hmacSha256(String raw, String salt) {
        try {
            Mac mac = Mac.getInstance("HmacSHA256");
            SecretKeySpec keySpec = new SecretKeySpec(
                    salt.getBytes(StandardCharsets.UTF_8),
                    "HmacSHA256"
            );
            mac.init(keySpec);
            byte[] digest = mac.doFinal(raw.getBytes(StandardCharsets.UTF_8));
            return HexFormat.of().formatHex(digest);
        } catch (Exception ex) {
            throw new IllegalStateException("Unable to generate checksum", ex);
        }
    }

    public static void main(String[] args) {
        String mid  = "B10001";
        String vpa  = "8898327678@upi";
        String salt = System.getenv("PG_SALT");

        String raw = String.join("|", mid, "VPA_VERIFY", vpa, salt);
        System.out.println(hmacSha256(raw, salt));
    }
}
```
</Tab>
<Tab title="Node.js">
```javascript
const crypto = require('crypto');

function generateChecksum(parts, salt) {
  const raw = [...parts, salt].join('|');
  return crypto.createHmac('sha256', salt).update(raw, 'utf8').digest('hex');
}

const checksum = generateChecksum(
  ['B10001', 'VPA_VERIFY', '8898327678@upi'],
  process.env.PG_SALT
);

console.log(checksum);
```
</Tab>
<Tab title="PHP">
```php
<?php
function generateChecksum(array $parts, string $salt): string {
    $raw = implode('|', array_merge($parts, [$salt]));
    return hash_hmac('sha256', $raw, $salt);
}

$checksum = generateChecksum(
    ['B10001', 'VPA_VERIFY', '8898327678@upi'],
    getenv('PG_SALT')
);

echo $checksum;
?>
```
</Tab>
<Tab title="Python">
```python
import hmac
import hashlib
import os

def generate_checksum(parts: list[str], salt: str) -> str:
    raw = "|".join([*parts, salt])
    return hmac.new(salt.encode(), raw.encode(), hashlib.sha256).hexdigest()

checksum = generate_checksum(
    ["B10001", "VPA_VERIFY", "8898327678@upi"],
    os.environ["PG_SALT"]
)

print(checksum)
```
</Tab>
<Tab title="C#">
```csharp
using System;
using System.Linq;
using System.Security.Cryptography;
using System.Text;

public static class ChecksumGenerator
{
    public static string Generate(string[] parts, string salt)
    {
        var raw = string.Join("|", parts.Concat(new[] { salt }));
        using var hmac = new HMACSHA256(Encoding.UTF8.GetBytes(salt));
        var hash = hmac.ComputeHash(Encoding.UTF8.GetBytes(raw));
        return Convert.ToHexString(hash).ToLowerInvariant();
    }

    public static void Main()
    {
        var checksum = Generate(
            new[] { "B10001", "VPA_VERIFY", "8898327678@upi" },
            Environment.GetEnvironmentVariable("PG_SALT")!
        );
        Console.WriteLine(checksum);
    }
}
```
</Tab>
</Tabs>

<br />

## Salt Handling Best Practices

1. Store `salt` in a managed secret store such as AWS Secrets Manager, Azure Key Vault, GCP Secret Manager, or HashiCorp Vault.
2. Rotate the salt during planned maintenance windows only after coordinating with the gateway operations team.
3. Maintain separate salts for UAT and production.
4. Never log the raw checksum string because it contains the salt.
5. Never send the salt in API payloads, query parameters, headers, or webhooks.

<br />

## Troubleshooting Checksum Failures

| Symptom | Cause | Resolution |
| --- | --- | --- |
| `401 Unauthorized` | Invalid checksum | Rebuild the raw string in the documented field order |
| Works in UAT, fails in prod | Using UAT salt in production | Load credentials by environment |
| Fails only for amounts | Amount formatting mismatch | Use the exact amount string sent in the request |
| Fails intermittently | Null or blank optional fields included inconsistently | Include only fields specified in the formula |
| Gateway says merchant invalid | Wrong `X-Merchant-Id` | Verify the merchant ID for the active environment |
Authenticate every S2S Payment Gateway request by generating and sending an HMAC-SHA256 checksum built from endpoint-specific request parameters.

## How Authentication Works

Each merchant receives a unique:

| Credential | Description | Storage Requirement |
| --- | --- | --- |
| `mid` | Merchant identifier assigned during onboarding | Server-side configuration |
| `salt` | Secret value used to generate HMAC-SHA256 checksums | Secret manager or encrypted environment variable |
| Encryption key | AES-256 key for request and response encryption | Secret manager only |
| IV | Initialization vector for AES-256-CBC | Secret manager only |

For every API request, your server:

1. Builds the raw checksum string using the API-specific formula.
2. Appends `salt` as the final component of the formula.
3. Generates an HMAC-SHA256 digest using `salt` as the signing key.
4. Sends the checksum in the `X-Checksum` header.

<Callout theme="danger" icon="fa-solid fa-shield-xmark">
Never generate checksums in browser JavaScript or mobile apps. The `salt` must remain only on your backend.
</Callout>

<br />

## Checksum Header

| Header | Required | Description |
| --- | --- | --- |
| `X-Merchant-Id` | Yes | Your merchant ID, for example `B10001` |
| `X-Checksum` | Yes | HMAC-SHA256 checksum generated from the endpoint formula |
| `Content-Type` | Yes for POST | Must be `application/json` |

<br />

## Checksum Formulas

| API | Method | Formula |
| --- | --- | --- |
| Verify VPA | `POST /vas/verify_vpa` | `mid\|VPA_VERIFY\|vpa\|salt` |
| Netbanking List | `GET /vas/netbanking` | `mid\|NETBANKING_LIST\|salt` |
| Enabled Instruments | `GET /vas/instruments` | `mid\|ENABLED_INSTRUMENTS\|salt` |
| Initiate Transaction | `POST /payment/initiate` | `mid\|orderNo\|rrn\|txnAmount\|paymentMode\|paymentCode\|salt` |
| Transaction Status | `POST /merchant/v1/getTxnStatus` | `mid\|rrn\|txnId\|salt` |

<br />

## Validation Rules

The gateway validates checksums using the same formula and merchant salt. A request is rejected when:

- The `X-Checksum` header is missing.
- The `X-Merchant-Id` header is invalid or inactive.
- A required field used in the formula is missing.
- Field ordering in the raw string is incorrect.
- Extra whitespace or case changes are introduced before hashing.
- The request was signed with the wrong environment's salt.

<br />

## Checksum Example — Verify VPA

Request payload:

```json
{
  "mid": "B10001",
  "vpa": "8898327678@upi"
}
```

Raw checksum string:

```text
B10001|VPA_VERIFY|8898327678@upi|your-secret-salt
```

Generated checksum:

```text
3d4f2a1b9c8e7f6a5b4c3d2e1f0a9b8c7d6e5f4a3b2c1d0e9f8a7b6c5d4e3f2
```

HTTP request:

```bash
curl -X POST https://<uat-host>:8085/s2s/v1/vas/verify_vpa \
  -H "Content-Type: application/json" \
  -H "X-Merchant-Id: B10001" \
  -H "X-Checksum: 3d4f2a1b9c8e7f6a5b4c3d2e1f0a9b8c7d6e5f4a3b2c1d0e9f8a7b6c5d4e3f2" \
  -d '{"encryptedData":"<encrypted-payload>"}'
```

<br />

## Code Samples

<Tabs>
<Tab title="Java">
```java
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;
import java.util.HexFormat;

public final class ChecksumGenerator {
    private ChecksumGenerator() {}

    public static String hmacSha256(String raw, String salt) {
        try {
            Mac mac = Mac.getInstance("HmacSHA256");
            SecretKeySpec keySpec = new SecretKeySpec(
                    salt.getBytes(StandardCharsets.UTF_8),
                    "HmacSHA256"
            );
            mac.init(keySpec);
            byte[] digest = mac.doFinal(raw.getBytes(StandardCharsets.UTF_8));
            return HexFormat.of().formatHex(digest);
        } catch (Exception ex) {
            throw new IllegalStateException("Unable to generate checksum", ex);
        }
    }

    public static void main(String[] args) {
        String mid  = "B10001";
        String vpa  = "8898327678@upi";
        String salt = System.getenv("PG_SALT");

        String raw = String.join("|", mid, "VPA_VERIFY", vpa, salt);
        System.out.println(hmacSha256(raw, salt));
    }
}
```
</Tab>
<Tab title="Node.js">
```javascript
const crypto = require('crypto');

function generateChecksum(parts, salt) {
  const raw = [...parts, salt].join('|');
  return crypto.createHmac('sha256', salt).update(raw, 'utf8').digest('hex');
}

const checksum = generateChecksum(
  ['B10001', 'VPA_VERIFY', '8898327678@upi'],
  process.env.PG_SALT
);

console.log(checksum);
```
</Tab>
<Tab title="PHP">
```php
<?php
function generateChecksum(array $parts, string $salt): string {
    $raw = implode('|', array_merge($parts, [$salt]));
    return hash_hmac('sha256', $raw, $salt);
}

$checksum = generateChecksum(
    ['B10001', 'VPA_VERIFY', '8898327678@upi'],
    getenv('PG_SALT')
);

echo $checksum;
?>
```
</Tab>
<Tab title="Python">
```python
import hmac
import hashlib
import os

def generate_checksum(parts: list[str], salt: str) -> str:
    raw = "|".join([*parts, salt])
    return hmac.new(salt.encode(), raw.encode(), hashlib.sha256).hexdigest()

checksum = generate_checksum(
    ["B10001", "VPA_VERIFY", "8898327678@upi"],
    os.environ["PG_SALT"]
)

print(checksum)
```
</Tab>
<Tab title="C#">
```csharp
using System;
using System.Linq;
using System.Security.Cryptography;
using System.Text;

public static class ChecksumGenerator
{
    public static string Generate(string[] parts, string salt)
    {
        var raw = string.Join("|", parts.Concat(new[] { salt }));
        using var hmac = new HMACSHA256(Encoding.UTF8.GetBytes(salt));
        var hash = hmac.ComputeHash(Encoding.UTF8.GetBytes(raw));
        return Convert.ToHexString(hash).ToLowerInvariant();
    }

    public static void Main()
    {
        var checksum = Generate(
            new[] { "B10001", "VPA_VERIFY", "8898327678@upi" },
            Environment.GetEnvironmentVariable("PG_SALT")!
        );
        Console.WriteLine(checksum);
    }
}
```
</Tab>
</Tabs>

<br />

## Salt Handling Best Practices

1. Store `salt` in a managed secret store such as AWS Secrets Manager, Azure Key Vault, GCP Secret Manager, or HashiCorp Vault.
2. Rotate the salt during planned maintenance windows only after coordinating with the gateway operations team.
3. Maintain separate salts for UAT and production.
4. Never log the raw checksum string because it contains the salt.
5. Never send the salt in API payloads, query parameters, headers, or webhooks.

<br />

## Troubleshooting Checksum Failures

| Symptom | Cause | Resolution |
| --- | --- | --- |
| `401 Unauthorized` | Invalid checksum | Rebuild the raw string in the documented field order |
| Works in UAT, fails in prod | Using UAT salt in production | Load credentials by environment |
| Fails only for amounts | Amount formatting mismatch | Use the exact amount string sent in the request |
| Fails intermittently | Null or blank optional fields included inconsistently | Include only fields specified in the formula |
| Gateway says merchant invalid | Wrong `X-Merchant-Id` | Verify the merchant ID for the active environment |
Authenticate every S2S Payment Gateway request by generating and sending an HMAC-SHA256 checksum built from endpoint-specific request parameters.

## How Authentication Works

Each merchant receives a unique:

| Credential | Description | Storage Requirement |
| --- | --- | --- |
| `mid` | Merchant identifier assigned during onboarding | Server-side configuration |
| `salt` | Secret value used to generate HMAC-SHA256 checksums | Secret manager or encrypted environment variable |
| Encryption key | AES-256 key for request and response encryption | Secret manager only |
| IV | Initialization vector for AES-256-CBC | Secret manager only |

For every API request, your server:

1. Builds the raw checksum string using the API-specific formula.
2. Appends `salt` as the final component of the formula.
3. Generates an HMAC-SHA256 digest using `salt` as the signing key.
4. Sends the checksum in the `X-Checksum` header.

<Callout theme="danger" icon="fa-solid fa-shield-xmark">
Never generate checksums in browser JavaScript or mobile apps. The `salt` must remain only on your backend.
</Callout>

<br />

## Checksum Header

| Header | Required | Description |
| --- | --- | --- |
| `X-Merchant-Id` | Yes | Your merchant ID, for example `B10001` |
| `X-Checksum` | Yes | HMAC-SHA256 checksum generated from the endpoint formula |
| `Content-Type` | Yes for POST | Must be `application/json` |

<br />

## Checksum Formulas

| API | Method | Formula |
| --- | --- | --- |
| Verify VPA | `POST /vas/verify_vpa` | `mid\|VPA_VERIFY\|vpa\|salt` |
| Netbanking List | `GET /vas/netbanking` | `mid\|NETBANKING_LIST\|salt` |
| Enabled Instruments | `GET /vas/instruments` | `mid\|ENABLED_INSTRUMENTS\|salt` |
| Initiate Transaction | `POST /payment/initiate` | `mid\|orderNo\|rrn\|txnAmount\|paymentMode\|paymentCode\|salt` |
| Transaction Status | `POST /merchant/v1/getTxnStatus` | `mid\|rrn\|txnId\|salt` |

<br />

## Validation Rules

The gateway validates checksums using the same formula and merchant salt. A request is rejected when:

- The `X-Checksum` header is missing.
- The `X-Merchant-Id` header is invalid or inactive.
- A required field used in the formula is missing.
- Field ordering in the raw string is incorrect.
- Extra whitespace or case changes are introduced before hashing.
- The request was signed with the wrong environment's salt.

<br />

## Checksum Example — Verify VPA

Request payload:

```json
{
  "mid": "B10001",
  "vpa": "8898327678@upi"
}
```

Raw checksum string:

```text
B10001|VPA_VERIFY|8898327678@upi|your-secret-salt
```

Generated checksum:

```text
3d4f2a1b9c8e7f6a5b4c3d2e1f0a9b8c7d6e5f4a3b2c1d0e9f8a7b6c5d4e3f2
```

HTTP request:

```bash
curl -X POST https://<uat-host>:8085/s2s/v1/vas/verify_vpa \
  -H "Content-Type: application/json" \
  -H "X-Merchant-Id: B10001" \
  -H "X-Checksum: 3d4f2a1b9c8e7f6a5b4c3d2e1f0a9b8c7d6e5f4a3b2c1d0e9f8a7b6c5d4e3f2" \
  -d '{"encryptedData":"<encrypted-payload>"}'
```

<br />

## Code Samples

<Tabs>
<Tab title="Java">
```java
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;
import java.util.HexFormat;

public final class ChecksumGenerator {
    private ChecksumGenerator() {}

    public static String hmacSha256(String raw, String salt) {
        try {
            Mac mac = Mac.getInstance("HmacSHA256");
            SecretKeySpec keySpec = new SecretKeySpec(
                    salt.getBytes(StandardCharsets.UTF_8),
                    "HmacSHA256"
            );
            mac.init(keySpec);
            byte[] digest = mac.doFinal(raw.getBytes(StandardCharsets.UTF_8));
            return HexFormat.of().formatHex(digest);
        } catch (Exception ex) {
            throw new IllegalStateException("Unable to generate checksum", ex);
        }
    }

    public static void main(String[] args) {
        String mid  = "B10001";
        String vpa  = "8898327678@upi";
        String salt = System.getenv("PG_SALT");

        String raw = String.join("|", mid, "VPA_VERIFY", vpa, salt);
        System.out.println(hmacSha256(raw, salt));
    }
}
```
</Tab>
<Tab title="Node.js">
```javascript
const crypto = require('crypto');

function generateChecksum(parts, salt) {
  const raw = [...parts, salt].join('|');
  return crypto.createHmac('sha256', salt).update(raw, 'utf8').digest('hex');
}

const checksum = generateChecksum(
  ['B10001', 'VPA_VERIFY', '8898327678@upi'],
  process.env.PG_SALT
);

console.log(checksum);
```
</Tab>
<Tab title="PHP">
```php
<?php
function generateChecksum(array $parts, string $salt): string {
    $raw = implode('|', array_merge($parts, [$salt]));
    return hash_hmac('sha256', $raw, $salt);
}

$checksum = generateChecksum(
    ['B10001', 'VPA_VERIFY', '8898327678@upi'],
    getenv('PG_SALT')
);

echo $checksum;
?>
```
</Tab>
<Tab title="Python">
```python
import hmac
import hashlib
import os

def generate_checksum(parts: list[str], salt: str) -> str:
    raw = "|".join([*parts, salt])
    return hmac.new(salt.encode(), raw.encode(), hashlib.sha256).hexdigest()

checksum = generate_checksum(
    ["B10001", "VPA_VERIFY", "8898327678@upi"],
    os.environ["PG_SALT"]
)

print(checksum)
```
</Tab>
<Tab title="C#">
```csharp
using System;
using System.Linq;
using System.Security.Cryptography;
using System.Text;

public static class ChecksumGenerator
{
    public static string Generate(string[] parts, string salt)
    {
        var raw = string.Join("|", parts.Concat(new[] { salt }));
        using var hmac = new HMACSHA256(Encoding.UTF8.GetBytes(salt));
        var hash = hmac.ComputeHash(Encoding.UTF8.GetBytes(raw));
        return Convert.ToHexString(hash).ToLowerInvariant();
    }

    public static void Main()
    {
        var checksum = Generate(
            new[] { "B10001", "VPA_VERIFY", "8898327678@upi" },
            Environment.GetEnvironmentVariable("PG_SALT")!
        );
        Console.WriteLine(checksum);
    }
}
```
</Tab>
</Tabs>

<br />

## Salt Handling Best Practices

1. Store `salt` in a managed secret store such as AWS Secrets Manager, Azure Key Vault, GCP Secret Manager, or HashiCorp Vault.
2. Rotate the salt during planned maintenance windows only after coordinating with the gateway operations team.
3. Maintain separate salts for UAT and production.
4. Never log the raw checksum string because it contains the salt.
5. Never send the salt in API payloads, query parameters, headers, or webhooks.

<br />

## Troubleshooting Checksum Failures

| Symptom | Cause | Resolution |
| --- | --- | --- |
| `401 Unauthorized` | Invalid checksum | Rebuild the raw string in the documented field order |
| Works in UAT, fails in prod | Using UAT salt in production | Load credentials by environment |
| Fails only for amounts | Amount formatting mismatch | Use the exact amount string sent in the request |
| Fails intermittently | Null or blank optional fields included inconsistently | Include only fields specified in the formula |
| Gateway says merchant invalid | Wrong `X-Merchant-Id` | Verify the merchant ID for the active environment |
Authenticate every S2S Payment Gateway request by generating and sending an HMAC-SHA256 checksum built from endpoint-specific request parameters.

## How Authentication Works

Each merchant receives a unique:

| Credential | Description | Storage Requirement |
| --- | --- | --- |
| `mid` | Merchant identifier assigned during onboarding | Server-side configuration |
| `salt` | Secret value used to generate HMAC-SHA256 checksums | Secret manager or encrypted environment variable |
| Encryption key | AES-256 key for request and response encryption | Secret manager only |
| IV | Initialization vector for AES-256-CBC | Secret manager only |

For every API request, your server:

1. Builds the raw checksum string using the API-specific formula.
2. Appends `salt` as the final component of the formula.
3. Generates an HMAC-SHA256 digest using `salt` as the signing key.
4. Sends the checksum in the `X-Checksum` header.

<Callout theme="danger" icon="fa-solid fa-shield-xmark">
Never generate checksums in browser JavaScript or mobile apps. The `salt` must remain only on your backend.
</Callout>

<br />

## Checksum Header

| Header | Required | Description |
| --- | --- | --- |
| `X-Merchant-Id` | Yes | Your merchant ID, for example `B10001` |
| `X-Checksum` | Yes | HMAC-SHA256 checksum generated from the endpoint formula |
| `Content-Type` | Yes for POST | Must be `application/json` |

<br />

## Checksum Formulas

| API | Method | Formula |
| --- | --- | --- |
| Verify VPA | `POST /vas/verify_vpa` | `mid\|VPA_VERIFY\|vpa\|salt` |
| Netbanking List | `GET /vas/netbanking` | `mid\|NETBANKING_LIST\|salt` |
| Enabled Instruments | `GET /vas/instruments` | `mid\|ENABLED_INSTRUMENTS\|salt` |
| Initiate Transaction | `POST /payment/initiate` | `mid\|orderNo\|rrn\|txnAmount\|paymentMode\|paymentCode\|salt` |
| Transaction Status | `POST /merchant/v1/getTxnStatus` | `mid\|rrn\|txnId\|salt` |

<br />

## Validation Rules

The gateway validates checksums using the same formula and merchant salt. A request is rejected when:

- The `X-Checksum` header is missing.
- The `X-Merchant-Id` header is invalid or inactive.
- A required field used in the formula is missing.
- Field ordering in the raw string is incorrect.
- Extra whitespace or case changes are introduced before hashing.
- The request was signed with the wrong environment's salt.

<br />

## Checksum Example — Verify VPA

Request payload:

```json
{
  "mid": "B10001",
  "vpa": "8898327678@upi"
}
```

Raw checksum string:

```text
B10001|VPA_VERIFY|8898327678@upi|your-secret-salt
```

Generated checksum:

```text
3d4f2a1b9c8e7f6a5b4c3d2e1f0a9b8c7d6e5f4a3b2c1d0e9f8a7b6c5d4e3f2
```

HTTP request:

```bash
curl -X POST https://<uat-host>:8085/s2s/v1/vas/verify_vpa \
  -H "Content-Type: application/json" \
  -H "X-Merchant-Id: B10001" \
  -H "X-Checksum: 3d4f2a1b9c8e7f6a5b4c3d2e1f0a9b8c7d6e5f4a3b2c1d0e9f8a7b6c5d4e3f2" \
  -d '{"encryptedData":"<encrypted-payload>"}'
```

<br />

## Code Samples

<Tabs>
<Tab title="Java">
```java
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;
import java.util.HexFormat;

public final class ChecksumGenerator {
    private ChecksumGenerator() {}

    public static String hmacSha256(String raw, String salt) {
        try {
            Mac mac = Mac.getInstance("HmacSHA256");
            SecretKeySpec keySpec = new SecretKeySpec(
                    salt.getBytes(StandardCharsets.UTF_8),
                    "HmacSHA256"
            );
            mac.init(keySpec);
            byte[] digest = mac.doFinal(raw.getBytes(StandardCharsets.UTF_8));
            return HexFormat.of().formatHex(digest);
        } catch (Exception ex) {
            throw new IllegalStateException("Unable to generate checksum", ex);
        }
    }

    public static void main(String[] args) {
        String mid  = "B10001";
        String vpa  = "8898327678@upi";
        String salt = System.getenv("PG_SALT");

        String raw = String.join("|", mid, "VPA_VERIFY", vpa, salt);
        System.out.println(hmacSha256(raw, salt));
    }
}
```
</Tab>
<Tab title="Node.js">
```javascript
const crypto = require('crypto');

function generateChecksum(parts, salt) {
  const raw = [...parts, salt].join('|');
  return crypto.createHmac('sha256', salt).update(raw, 'utf8').digest('hex');
}

const checksum = generateChecksum(
  ['B10001', 'VPA_VERIFY', '8898327678@upi'],
  process.env.PG_SALT
);

console.log(checksum);
```
</Tab>
<Tab title="PHP">
```php
<?php
function generateChecksum(array $parts, string $salt): string {
    $raw = implode('|', array_merge($parts, [$salt]));
    return hash_hmac('sha256', $raw, $salt);
}

$checksum = generateChecksum(
    ['B10001', 'VPA_VERIFY', '8898327678@upi'],
    getenv('PG_SALT')
);

echo $checksum;
?>
```
</Tab>
<Tab title="Python">
```python
import hmac
import hashlib
import os

def generate_checksum(parts: list[str], salt: str) -> str:
    raw = "|".join([*parts, salt])
    return hmac.new(salt.encode(), raw.encode(), hashlib.sha256).hexdigest()

checksum = generate_checksum(
    ["B10001", "VPA_VERIFY", "8898327678@upi"],
    os.environ["PG_SALT"]
)

print(checksum)
```
</Tab>
<Tab title="C#">
```csharp
using System;
using System.Linq;
using System.Security.Cryptography;
using System.Text;

public static class ChecksumGenerator
{
    public static string Generate(string[] parts, string salt)
    {
        var raw = string.Join("|", parts.Concat(new[] { salt }));
        using var hmac = new HMACSHA256(Encoding.UTF8.GetBytes(salt));
        var hash = hmac.ComputeHash(Encoding.UTF8.GetBytes(raw));
        return Convert.ToHexString(hash).ToLowerInvariant();
    }

    public static void Main()
    {
        var checksum = Generate(
            new[] { "B10001", "VPA_VERIFY", "8898327678@upi" },
            Environment.GetEnvironmentVariable("PG_SALT")!
        );
        Console.WriteLine(checksum);
    }
}
```
</Tab>
</Tabs>

<br />

## Salt Handling Best Practices

1. Store `salt` in a managed secret store such as AWS Secrets Manager, Azure Key Vault, GCP Secret Manager, or HashiCorp Vault.
2. Rotate the salt during planned maintenance windows only after coordinating with the gateway operations team.
3. Maintain separate salts for UAT and production.
4. Never log the raw checksum string because it contains the salt.
5. Never send the salt in API payloads, query parameters, headers, or webhooks.

<br />

## Troubleshooting Checksum Failures

| Symptom | Cause | Resolution |
| --- | --- | --- |
| `401 Unauthorized` | Invalid checksum | Rebuild the raw string in the documented field order |
| Works in UAT, fails in prod | Using UAT salt in production | Load credentials by environment |
| Fails only for amounts | Amount formatting mismatch | Use the exact amount string sent in the request |
| Fails intermittently | Null or blank optional fields included inconsistently | Include only fields specified in the formula |
| Gateway says merchant invalid | Wrong `X-Merchant-Id` | Verify the merchant ID for the active environment |
Authenticate every S2S Payment Gateway request by generating and sending an HMAC-SHA256 checksum built from endpoint-specific request parameters.

## How Authentication Works

Each merchant receives a unique:

| Credential | Description | Storage Requirement |
| --- | --- | --- |
| `mid` | Merchant identifier assigned during onboarding | Server-side configuration |
| `salt` | Secret value used to generate HMAC-SHA256 checksums | Secret manager or encrypted environment variable |
| Encryption key | AES-256 key for request and response encryption | Secret manager only |
| IV | Initialization vector for AES-256-CBC | Secret manager only |

For every API request, your server:

1. Builds the raw checksum string using the API-specific formula.
2. Appends `salt` as the final component of the formula.
3. Generates an HMAC-SHA256 digest using `salt` as the signing key.
4. Sends the checksum in the `X-Checksum` header.

<Callout theme="danger" icon="fa-solid fa-shield-xmark">
Never generate checksums in browser JavaScript or mobile apps. The `salt` must remain only on your backend.
</Callout>

<br />

## Checksum Header

| Header | Required | Description |
| --- | --- | --- |
| `X-Merchant-Id` | Yes | Your merchant ID, for example `B10001` |
| `X-Checksum` | Yes | HMAC-SHA256 checksum generated from the endpoint formula |
| `Content-Type` | Yes for POST | Must be `application/json` |

<br />

## Checksum Formulas

| API | Method | Formula |
| --- | --- | --- |
| Verify VPA | `POST /vas/verify_vpa` | `mid\|VPA_VERIFY\|vpa\|salt` |
| Netbanking List | `GET /vas/netbanking` | `mid\|NETBANKING_LIST\|salt` |
| Enabled Instruments | `GET /vas/instruments` | `mid\|ENABLED_INSTRUMENTS\|salt` |
| Initiate Transaction | `POST /payment/initiate` | `mid\|orderNo\|rrn\|txnAmount\|paymentMode\|paymentCode\|salt` |
| Transaction Status | `POST /merchant/v1/getTxnStatus` | `mid\|rrn\|txnId\|salt` |

<br />

## Validation Rules

The gateway validates checksums using the same formula and merchant salt. A request is rejected when:

- The `X-Checksum` header is missing.
- The `X-Merchant-Id` header is invalid or inactive.
- A required field used in the formula is missing.
- Field ordering in the raw string is incorrect.
- Extra whitespace or case changes are introduced before hashing.
- The request was signed with the wrong environment's salt.

<br />

## Checksum Example — Verify VPA

Request payload:

```json
{
  "mid": "B10001",
  "vpa": "8898327678@upi"
}
```

Raw checksum string:

```text
B10001|VPA_VERIFY|8898327678@upi|your-secret-salt
```

Generated checksum:

```text
3d4f2a1b9c8e7f6a5b4c3d2e1f0a9b8c7d6e5f4a3b2c1d0e9f8a7b6c5d4e3f2
```

HTTP request:

```bash
curl -X POST https://<uat-host>:8085/s2s/v1/vas/verify_vpa \
  -H "Content-Type: application/json" \
  -H "X-Merchant-Id: B10001" \
  -H "X-Checksum: 3d4f2a1b9c8e7f6a5b4c3d2e1f0a9b8c7d6e5f4a3b2c1d0e9f8a7b6c5d4e3f2" \
  -d '{"encryptedData":"<encrypted-payload>"}'
```

<br />

## Code Samples

<Tabs>
<Tab title="Java">
```java
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;
import java.util.HexFormat;

public final class ChecksumGenerator {
    private ChecksumGenerator() {}

    public static String hmacSha256(String raw, String salt) {
        try {
            Mac mac = Mac.getInstance("HmacSHA256");
            SecretKeySpec keySpec = new SecretKeySpec(
                    salt.getBytes(StandardCharsets.UTF_8),
                    "HmacSHA256"
            );
            mac.init(keySpec);
            byte[] digest = mac.doFinal(raw.getBytes(StandardCharsets.UTF_8));
            return HexFormat.of().formatHex(digest);
        } catch (Exception ex) {
            throw new IllegalStateException("Unable to generate checksum", ex);
        }
    }

    public static void main(String[] args) {
        String mid  = "B10001";
        String vpa  = "8898327678@upi";
        String salt = System.getenv("PG_SALT");

        String raw = String.join("|", mid, "VPA_VERIFY", vpa, salt);
        System.out.println(hmacSha256(raw, salt));
    }
}
```
</Tab>
<Tab title="Node.js">
```javascript
const crypto = require('crypto');

function generateChecksum(parts, salt) {
  const raw = [...parts, salt].join('|');
  return crypto.createHmac('sha256', salt).update(raw, 'utf8').digest('hex');
}

const checksum = generateChecksum(
  ['B10001', 'VPA_VERIFY', '8898327678@upi'],
  process.env.PG_SALT
);

console.log(checksum);
```
</Tab>
<Tab title="PHP">
```php
<?php
function generateChecksum(array $parts, string $salt): string {
    $raw = implode('|', array_merge($parts, [$salt]));
    return hash_hmac('sha256', $raw, $salt);
}

$checksum = generateChecksum(
    ['B10001', 'VPA_VERIFY', '8898327678@upi'],
    getenv('PG_SALT')
);

echo $checksum;
?>
```
</Tab>
<Tab title="Python">
```python
import hmac
import hashlib
import os

def generate_checksum(parts: list[str], salt: str) -> str:
    raw = "|".join([*parts, salt])
    return hmac.new(salt.encode(), raw.encode(), hashlib.sha256).hexdigest()

checksum = generate_checksum(
    ["B10001", "VPA_VERIFY", "8898327678@upi"],
    os.environ["PG_SALT"]
)

print(checksum)
```
</Tab>
<Tab title="C#">
```csharp
using System;
using System.Linq;
using System.Security.Cryptography;
using System.Text;

public static class ChecksumGenerator
{
    public static string Generate(string[] parts, string salt)
    {
        var raw = string.Join("|", parts.Concat(new[] { salt }));
        using var hmac = new HMACSHA256(Encoding.UTF8.GetBytes(salt));
        var hash = hmac.ComputeHash(Encoding.UTF8.GetBytes(raw));
        return Convert.ToHexString(hash).ToLowerInvariant();
    }

    public static void Main()
    {
        var checksum = Generate(
            new[] { "B10001", "VPA_VERIFY", "8898327678@upi" },
            Environment.GetEnvironmentVariable("PG_SALT")!
        );
        Console.WriteLine(checksum);
    }
}
```
</Tab>
</Tabs>

<br />

## Salt Handling Best Practices

1. Store `salt` in a managed secret store such as AWS Secrets Manager, Azure Key Vault, GCP Secret Manager, or HashiCorp Vault.
2. Rotate the salt during planned maintenance windows only after coordinating with the gateway operations team.
3. Maintain separate salts for UAT and production.
4. Never log the raw checksum string because it contains the salt.
5. Never send the salt in API payloads, query parameters, headers, or webhooks.

<br />

## Troubleshooting Checksum Failures

| Symptom | Cause | Resolution |
| --- | --- | --- |
| `401 Unauthorized` | Invalid checksum | Rebuild the raw string in the documented field order |
| Works in UAT, fails in prod | Using UAT salt in production | Load credentials by environment |
| Fails only for amounts | Amount formatting mismatch | Use the exact amount string sent in the request |
| Fails intermittently | Null or blank optional fields included inconsistently | Include only fields specified in the formula |
| Gateway says merchant invalid | Wrong `X-Merchant-Id` | Verify the merchant ID for the active environment |
Authenticate every S2S Payment Gateway request by generating and sending an HMAC-SHA256 checksum built from endpoint-specific request parameters.

## How Authentication Works

Each merchant receives a unique:

| Credential | Description | Storage Requirement |
| --- | --- | --- |
| `mid` | Merchant identifier assigned during onboarding | Server-side configuration |
| `salt` | Secret value used to generate HMAC-SHA256 checksums | Secret manager or encrypted environment variable |
| Encryption key | AES-256 key for request and response encryption | Secret manager only |
| IV | Initialization vector for AES-256-CBC | Secret manager only |

For every API request, your server:

1. Builds the raw checksum string using the API-specific formula.
2. Appends `salt` as the final component of the formula.
3. Generates an HMAC-SHA256 digest using `salt` as the signing key.
4. Sends the checksum in the `X-Checksum` header.

<Callout theme="danger" icon="fa-solid fa-shield-xmark">
Never generate checksums in browser JavaScript or mobile apps. The `salt` must remain only on your backend.
</Callout>

<br />

## Checksum Header

| Header | Required | Description |
| --- | --- | --- |
| `X-Merchant-Id` | Yes | Your merchant ID, for example `B10001` |
| `X-Checksum` | Yes | HMAC-SHA256 checksum generated from the endpoint formula |
| `Content-Type` | Yes for POST | Must be `application/json` |

<br />

## Checksum Formulas

| API | Method | Formula |
| --- | --- | --- |
| Verify VPA | `POST /vas/verify_vpa` | `mid\|VPA_VERIFY\|vpa\|salt` |
| Netbanking List | `GET /vas/netbanking` | `mid\|NETBANKING_LIST\|salt` |
| Enabled Instruments | `GET /vas/instruments` | `mid\|ENABLED_INSTRUMENTS\|salt` |
| Initiate Transaction | `POST /payment/initiate` | `mid\|orderNo\|rrn\|txnAmount\|paymentMode\|paymentCode\|salt` |
| Transaction Status | `POST /merchant/v1/getTxnStatus` | `mid\|rrn\|txnId\|salt` |

<br />

## Validation Rules

The gateway validates checksums using the same formula and merchant salt. A request is rejected when:

- The `X-Checksum` header is missing.
- The `X-Merchant-Id` header is invalid or inactive.
- A required field used in the formula is missing.
- Field ordering in the raw string is incorrect.
- Extra whitespace or case changes are introduced before hashing.
- The request was signed with the wrong environment's salt.

<br />

## Checksum Example — Verify VPA

Request payload:

```json
{
  "mid": "B10001",
  "vpa": "8898327678@upi"
}
```

Raw checksum string:

```text
B10001|VPA_VERIFY|8898327678@upi|your-secret-salt
```

Generated checksum:

```text
3d4f2a1b9c8e7f6a5b4c3d2e1f0a9b8c7d6e5f4a3b2c1d0e9f8a7b6c5d4e3f2
```

HTTP request:

```bash
curl -X POST https://<uat-host>:8085/s2s/v1/vas/verify_vpa \
  -H "Content-Type: application/json" \
  -H "X-Merchant-Id: B10001" \
  -H "X-Checksum: 3d4f2a1b9c8e7f6a5b4c3d2e1f0a9b8c7d6e5f4a3b2c1d0e9f8a7b6c5d4e3f2" \
  -d '{"encryptedData":"<encrypted-payload>"}'
```

<br />

## Code Samples

<Tabs>
<Tab title="Java">
```java
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;
import java.util.HexFormat;

public final class ChecksumGenerator {
    private ChecksumGenerator() {}

    public static String hmacSha256(String raw, String salt) {
        try {
            Mac mac = Mac.getInstance("HmacSHA256");
            SecretKeySpec keySpec = new SecretKeySpec(
                    salt.getBytes(StandardCharsets.UTF_8),
                    "HmacSHA256"
            );
            mac.init(keySpec);
            byte[] digest = mac.doFinal(raw.getBytes(StandardCharsets.UTF_8));
            return HexFormat.of().formatHex(digest);
        } catch (Exception ex) {
            throw new IllegalStateException("Unable to generate checksum", ex);
        }
    }

    public static void main(String[] args) {
        String mid  = "B10001";
        String vpa  = "8898327678@upi";
        String salt = System.getenv("PG_SALT");

        String raw = String.join("|", mid, "VPA_VERIFY", vpa, salt);
        System.out.println(hmacSha256(raw, salt));
    }
}
```
</Tab>
<Tab title="Node.js">
```javascript
const crypto = require('crypto');

function generateChecksum(parts, salt) {
  const raw = [...parts, salt].join('|');
  return crypto.createHmac('sha256', salt).update(raw, 'utf8').digest('hex');
}

const checksum = generateChecksum(
  ['B10001', 'VPA_VERIFY', '8898327678@upi'],
  process.env.PG_SALT
);

console.log(checksum);
```
</Tab>
<Tab title="PHP">
```php
<?php
function generateChecksum(array $parts, string $salt): string {
    $raw = implode('|', array_merge($parts, [$salt]));
    return hash_hmac('sha256', $raw, $salt);
}

$checksum = generateChecksum(
    ['B10001', 'VPA_VERIFY', '8898327678@upi'],
    getenv('PG_SALT')
);

echo $checksum;
?>
```
</Tab>
<Tab title="Python">
```python
import hmac
import hashlib
import os

def generate_checksum(parts: list[str], salt: str) -> str:
    raw = "|".join([*parts, salt])
    return hmac.new(salt.encode(), raw.encode(), hashlib.sha256).hexdigest()

checksum = generate_checksum(
    ["B10001", "VPA_VERIFY", "8898327678@upi"],
    os.environ["PG_SALT"]
)

print(checksum)
```
</Tab>
<Tab title="C#">
```csharp
using System;
using System.Linq;
using System.Security.Cryptography;
using System.Text;

public static class ChecksumGenerator
{
    public static string Generate(string[] parts, string salt)
    {
        var raw = string.Join("|", parts.Concat(new[] { salt }));
        using var hmac = new HMACSHA256(Encoding.UTF8.GetBytes(salt));
        var hash = hmac.ComputeHash(Encoding.UTF8.GetBytes(raw));
        return Convert.ToHexString(hash).ToLowerInvariant();
    }

    public static void Main()
    {
        var checksum = Generate(
            new[] { "B10001", "VPA_VERIFY", "8898327678@upi" },
            Environment.GetEnvironmentVariable("PG_SALT")!
        );
        Console.WriteLine(checksum);
    }
}
```
</Tab>
</Tabs>

<br />

## Salt Handling Best Practices

1. Store `salt` in a managed secret store such as AWS Secrets Manager, Azure Key Vault, GCP Secret Manager, or HashiCorp Vault.
2. Rotate the salt during planned maintenance windows only after coordinating with the gateway operations team.
3. Maintain separate salts for UAT and production.
4. Never log the raw checksum string because it contains the salt.
5. Never send the salt in API payloads, query parameters, headers, or webhooks.

<br />

## Troubleshooting Checksum Failures

| Symptom | Cause | Resolution |
| --- | --- | --- |
| `401 Unauthorized` | Invalid checksum | Rebuild the raw string in the documented field order |
| Works in UAT, fails in prod | Using UAT salt in production | Load credentials by environment |
| Fails only for amounts | Amount formatting mismatch | Use the exact amount string sent in the request |
| Fails intermittently | Null or blank optional fields included inconsistently | Include only fields specified in the formula |
| Gateway says merchant invalid | Wrong `X-Merchant-Id` | Verify the merchant ID for the active environment |