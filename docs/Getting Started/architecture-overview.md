---
title: '# Architecture Overview'
excerpt: >-
  High-level architecture of the S2S Payment Gateway — components, request
  lifecycle, and security layers.  Understand how the S2S Payment Gateway is
  structured, how your server communicates with the gateway, and how payment
  flows traverse the system.
deprecated: false
hidden: false
metadata:
  robots: index
---
# System Architecture

The S2S (Server-to-Server) Payment Gateway operates as a secure intermediary between your application server and the underlying payment networks (UPI, NPCI, card networks, bank portals).

```
┌─────────────────────────────────────────────────────────────────────┐
│                        MERCHANT ECOSYSTEM                           │
│                                                                     │
│   ┌──────────────┐      ┌──────────────┐     ┌──────────────────┐  │
│   │  Customer    │◄────►│  Merchant    │────►│  Merchant Server │  │
│   │  Browser /   │      │  Frontend    │     │  (Your Backend)  │  │
│   │  Mobile App  │      │  (Web/App)   │     └────────┬─────────┘  │
│   └──────────────┘      └──────────────┘              │            │
└───────────────────────────────────────────────────────┼────────────┘
                                                         │
                                              HTTPS + AES-256 + HMAC
                                                         │
┌───────────────────────────────────────────────────────▼────────────┐
│                     S2S PAYMENT GATEWAY                             │
│                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐  │
│  │  API Layer   │  │  Auth Layer  │  │  Encryption / Decryption │  │
│  │  (REST APIs) │  │  (HMAC-SHA256│  │  (AES-256-CBC)           │  │
│  └──────┬───────┘  └──────────────┘  └──────────────────────────┘  │
│         │                                                           │
│  ┌──────▼──────────────────────────────────────────────────────┐   │
│  │               PAYMENT ROUTING ENGINE                        │   │
│  │  UPI Handler │ NB Handler │ Card Handler │ Checkout Engine  │   │
│  └──────┬───────────┬────────────┬────────────────┬────────────┘   │
└─────────┼───────────┼────────────┼────────────────┼────────────────┘
          │           │            │                │
    ┌─────▼──┐  ┌─────▼──┐  ┌─────▼──┐      ┌─────▼──┐
    │  NPCI  │  │  Bank  │  │  Card  │      │Payment │
    │  UPI   │  │  Net   │  │Network │      │  Link  │
    │  Rails │  │Banking │  │(Visa/  │      │  Page  │
    └────────┘  └────────┘  │  MC)   │      └────────┘
                            └────────┘
```

<br />

## Request Lifecycle

Every API call follows this security pipeline:

1. **Payload construction** — Your server builds the JSON request body.
2. **AES-256 encryption** — The JSON body is encrypted using AES-256-CBC with your merchant-specific key and IV.
3. **Checksum generation** — An HMAC-SHA256 hash is computed from the pipe-delimited formula for the specific API.
4. **HTTP request** — The encrypted payload is sent over HTTPS with `X-Merchant-Id` and `X-Checksum` headers.
5. **Gateway validation** — The gateway decrypts the payload, validates the checksum, and processes the request.
6. **Encrypted response** — The gateway returns an AES-256-encrypted response body.
7. **Response decryption** — Your server decrypts the response using the same key and IV.
8. **Status evaluation** — Your application reads `response.status` and `response.statusCode` to determine the outcome.

<br />

## Core Components

| Component                        | Responsibility                                                           |
| -------------------------------- | ------------------------------------------------------------------------ |
| **VAS API** (`/vas/*`)           | Value-Added Services: VPA verification, instrument listing, bank listing |
| **Payment API** (`/payment/*`)   | Transaction initiation across all payment modes                          |
| **Merchant API** (`/merchant/*`) | Transaction status queries                                               |
| **Checkout API** (`/checkout/*`) | Hosted checkout sessions and payment link generation                     |
| **Webhook Engine**               | Asynchronous payment status notifications to your server                 |

<br />

## Security Architecture

The gateway enforces three independent security controls on every request:

<Columns layout="auto">
<Column>

**Transport Security**

* All communication over HTTPS/TLS 1.2+
* Certificate pinning recommended for mobile SDKs
* No plaintext HTTP allowed in production

**Authentication**

* HMAC-SHA256 checksum per request
* Formula varies by API endpoint
* `salt` is a shared secret — never client-side

</Column>
<Column>

**Payload Encryption**

* AES-256-CBC on request bodies
* AES-256-CBC on response bodies
* Unique key + IV pair per merchant

**Request Integrity**

* Checksum covers key transaction parameters
* Prevents parameter tampering in transit
* Gateway rejects any request with checksum mismatch

</Column>
</Columns>

<br />

## Data Flow by Payment Mode

| Payment Mode        | Customer Interaction         | Redirect Required  |
| ------------------- | ---------------------------- | ------------------ |
| UPI Collect         | Customer approves in UPI app | No                 |
| UPI Intent          | Deep-link to UPI app         | App switch         |
| Dynamic QR          | Customer scans QR code       | No                 |
| Net Banking         | Customer logs in at bank     | Yes (bank portal)  |
| Credit / Debit Card | Customer enters card details | Optional (3DS)     |
| Hosted Checkout     | Gateway-hosted page          | Yes (gateway page) |
| Payment Link        | Customer opens link          | Yes (gateway page) |

<br />

## Idempotency and Order Numbers

The `orderNo` field is your idempotency key. The gateway de-duplicates requests sharing the same `orderNo` within a configurable window. Rules:

- Generate `orderNo` on your server — never client-side.
- Use a globally unique identifier (UUID v4, or a prefixed timestamp+random combination).
- Store `orderNo` in your database **before** sending the API request so you can reconcile even if your server crashes after the gateway responds.
- Do not reuse `orderNo` for a new payment attempt; create a new order number.

> fa-solid fa-circle-info
>
> See the [Transaction Reconciliation](/docs/transaction-reconciliation) guide for the recommended database schema and reconciliation workflow.

Understand how the S2S Payment Gateway is structured, how your server communicates with the gateway, and how payment flows traverse the system.

## System Architecture

The S2S (Server-to-Server) Payment Gateway operates as a secure intermediary between your application server and the underlying payment networks (UPI, NPCI, card networks, bank portals).

```
┌─────────────────────────────────────────────────────────────────────┐
│                        MERCHANT ECOSYSTEM                           │
│                                                                     │
│   ┌──────────────┐      ┌──────────────┐     ┌──────────────────┐  │
│   │  Customer    │◄────►│  Merchant    │────►│  Merchant Server │  │
│   │  Browser /   │      │  Frontend    │     │  (Your Backend)  │  │
│   │  Mobile App  │      │  (Web/App)   │     └────────┬─────────┘  │
│   └──────────────┘      └──────────────┘              │            │
└───────────────────────────────────────────────────────┼────────────┘
                                                         │
                                              HTTPS + AES-256 + HMAC
                                                         │
┌───────────────────────────────────────────────────────▼────────────┐
│                     S2S PAYMENT GATEWAY                             │
│                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐  │
│  │  API Layer   │  │  Auth Layer  │  │  Encryption / Decryption │  │
│  │  (REST APIs) │  │  (HMAC-SHA256│  │  (AES-256-CBC)           │  │
│  └──────┬───────┘  └──────────────┘  └──────────────────────────┘  │
│         │                                                           │
│  ┌──────▼──────────────────────────────────────────────────────┐   │
│  │               PAYMENT ROUTING ENGINE                        │   │
│  │  UPI Handler │ NB Handler │ Card Handler │ Checkout Engine  │   │
│  └──────┬───────────┬────────────┬────────────────┬────────────┘   │
└─────────┼───────────┼────────────┼────────────────┼────────────────┘
          │           │            │                │
    ┌─────▼──┐  ┌─────▼──┐  ┌─────▼──┐      ┌─────▼──┐
    │  NPCI  │  │  Bank  │  │  Card  │      │Payment │
    │  UPI   │  │  Net   │  │Network │      │  Link  │
    │  Rails │  │Banking │  │(Visa/  │      │  Page  │
    └────────┘  └────────┘  │  MC)   │      └────────┘
                            └────────┘
```

<br />

## Request Lifecycle

Every API call follows this security pipeline:

1. **Payload construction** — Your server builds the JSON request body.
2. **AES-256 encryption** — The JSON body is encrypted using AES-256-CBC with your merchant-specific key and IV.
3. **Checksum generation** — An HMAC-SHA256 hash is computed from the pipe-delimited formula for the specific API.
4. **HTTP request** — The encrypted payload is sent over HTTPS with `X-Merchant-Id` and `X-Checksum` headers.
5. **Gateway validation** — The gateway decrypts the payload, validates the checksum, and processes the request.
6. **Encrypted response** — The gateway returns an AES-256-encrypted response body.
7. **Response decryption** — Your server decrypts the response using the same key and IV.
8. **Status evaluation** — Your application reads `response.status` and `response.statusCode` to determine the outcome.

<br />

## Core Components

| Component                        | Responsibility                                                           |
| -------------------------------- | ------------------------------------------------------------------------ |
| **VAS API** (`/vas/*`)           | Value-Added Services: VPA verification, instrument listing, bank listing |
| **Payment API** (`/payment/*`)   | Transaction initiation across all payment modes                          |
| **Merchant API** (`/merchant/*`) | Transaction status queries                                               |
| **Checkout API** (`/checkout/*`) | Hosted checkout sessions and payment link generation                     |
| **Webhook Engine**               | Asynchronous payment status notifications to your server                 |

<br />

## Security Architecture

The gateway enforces three independent security controls on every request:

<Columns layout="auto">
<Column>

**Transport Security**

* All communication over HTTPS/TLS 1.2+
* Certificate pinning recommended for mobile SDKs
* No plaintext HTTP allowed in production

**Authentication**

* HMAC-SHA256 checksum per request
* Formula varies by API endpoint
* `salt` is a shared secret — never client-side

</Column>
<Column>

**Payload Encryption**

* AES-256-CBC on request bodies
* AES-256-CBC on response bodies
* Unique key + IV pair per merchant

**Request Integrity**

* Checksum covers key transaction parameters
* Prevents parameter tampering in transit
* Gateway rejects any request with checksum mismatch

</Column>
</Columns>

<br />

## Data Flow by Payment Mode

| Payment Mode        | Customer Interaction         | Redirect Required  |
| ------------------- | ---------------------------- | ------------------ |
| UPI Collect         | Customer approves in UPI app | No                 |
| UPI Intent          | Deep-link to UPI app         | App switch         |
| Dynamic QR          | Customer scans QR code       | No                 |
| Net Banking         | Customer logs in at bank     | Yes (bank portal)  |
| Credit / Debit Card | Customer enters card details | Optional (3DS)     |
| Hosted Checkout     | Gateway-hosted page          | Yes (gateway page) |
| Payment Link        | Customer opens link          | Yes (gateway page) |

<br />

## Idempotency and Order Numbers

The `orderNo` field is your idempotency key. The gateway de-duplicates requests sharing the same `orderNo` within a configurable window. Rules:

- Generate `orderNo` on your server — never client-side.
- Use a globally unique identifier (UUID v4, or a prefixed timestamp+random combination).
- Store `orderNo` in your database **before** sending the API request so you can reconcile even if your server crashes after the gateway responds.
- Do not reuse `orderNo` for a new payment attempt; create a new order number.

> fa-solid fa-circle-info
>
> See the [Transaction Reconciliation](/docs/transaction-reconciliation) guide for the recommended database schema and reconciliation workflow.

Understand how the S2S Payment Gateway is structured, how your server communicates with the gateway, and how payment flows traverse the system.

## System Architecture

The S2S (Server-to-Server) Payment Gateway operates as a secure intermediary between your application server and the underlying payment networks (UPI, NPCI, card networks, bank portals).

```
┌─────────────────────────────────────────────────────────────────────┐
│                        MERCHANT ECOSYSTEM                           │
│                                                                     │
│   ┌──────────────┐      ┌──────────────┐     ┌──────────────────┐  │
│   │  Customer    │◄────►│  Merchant    │────►│  Merchant Server │  │
│   │  Browser /   │      │  Frontend    │     │  (Your Backend)  │  │
│   │  Mobile App  │      │  (Web/App)   │     └────────┬─────────┘  │
│   └──────────────┘      └──────────────┘              │            │
└───────────────────────────────────────────────────────┼────────────┘
                                                         │
                                              HTTPS + AES-256 + HMAC
                                                         │
┌───────────────────────────────────────────────────────▼────────────┐
│                     S2S PAYMENT GATEWAY                             │
│                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐  │
│  │  API Layer   │  │  Auth Layer  │  │  Encryption / Decryption │  │
│  │  (REST APIs) │  │  (HMAC-SHA256│  │  (AES-256-CBC)           │  │
│  └──────┬───────┘  └──────────────┘  └──────────────────────────┘  │
│         │                                                           │
│  ┌──────▼──────────────────────────────────────────────────────┐   │
│  │               PAYMENT ROUTING ENGINE                        │   │
│  │  UPI Handler │ NB Handler │ Card Handler │ Checkout Engine  │   │
│  └──────┬───────────┬────────────┬────────────────┬────────────┘   │
└─────────┼───────────┼────────────┼────────────────┼────────────────┘
          │           │            │                │
    ┌─────▼──┐  ┌─────▼──┐  ┌─────▼──┐      ┌─────▼──┐
    │  NPCI  │  │  Bank  │  │  Card  │      │Payment │
    │  UPI   │  │  Net   │  │Network │      │  Link  │
    │  Rails │  │Banking │  │(Visa/  │      │  Page  │
    └────────┘  └────────┘  │  MC)   │      └────────┘
                            └────────┘
```

<br />

## Request Lifecycle

Every API call follows this security pipeline:

1. **Payload construction** — Your server builds the JSON request body.
2. **AES-256 encryption** — The JSON body is encrypted using AES-256-CBC with your merchant-specific key and IV.
3. **Checksum generation** — An HMAC-SHA256 hash is computed from the pipe-delimited formula for the specific API.
4. **HTTP request** — The encrypted payload is sent over HTTPS with `X-Merchant-Id` and `X-Checksum` headers.
5. **Gateway validation** — The gateway decrypts the payload, validates the checksum, and processes the request.
6. **Encrypted response** — The gateway returns an AES-256-encrypted response body.
7. **Response decryption** — Your server decrypts the response using the same key and IV.
8. **Status evaluation** — Your application reads `response.status` and `response.statusCode` to determine the outcome.

<br />

## Core Components

| Component                        | Responsibility                                                           |
| -------------------------------- | ------------------------------------------------------------------------ |
| **VAS API** (`/vas/*`)           | Value-Added Services: VPA verification, instrument listing, bank listing |
| **Payment API** (`/payment/*`)   | Transaction initiation across all payment modes                          |
| **Merchant API** (`/merchant/*`) | Transaction status queries                                               |
| **Checkout API** (`/checkout/*`) | Hosted checkout sessions and payment link generation                     |
| **Webhook Engine**               | Asynchronous payment status notifications to your server                 |

<br />

## Security Architecture

The gateway enforces three independent security controls on every request:

<Columns layout="auto">
<Column>

**Transport Security**

* All communication over HTTPS/TLS 1.2+
* Certificate pinning recommended for mobile SDKs
* No plaintext HTTP allowed in production

**Authentication**

* HMAC-SHA256 checksum per request
* Formula varies by API endpoint
* `salt` is a shared secret — never client-side

</Column>
<Column>

**Payload Encryption**

* AES-256-CBC on request bodies
* AES-256-CBC on response bodies
* Unique key + IV pair per merchant

**Request Integrity**

* Checksum covers key transaction parameters
* Prevents parameter tampering in transit
* Gateway rejects any request with checksum mismatch

</Column>
</Columns>

<br />

## Data Flow by Payment Mode

| Payment Mode        | Customer Interaction         | Redirect Required  |
| ------------------- | ---------------------------- | ------------------ |
| UPI Collect         | Customer approves in UPI app | No                 |
| UPI Intent          | Deep-link to UPI app         | App switch         |
| Dynamic QR          | Customer scans QR code       | No                 |
| Net Banking         | Customer logs in at bank     | Yes (bank portal)  |
| Credit / Debit Card | Customer enters card details | Optional (3DS)     |
| Hosted Checkout     | Gateway-hosted page          | Yes (gateway page) |
| Payment Link        | Customer opens link          | Yes (gateway page) |

<br />

## Idempotency and Order Numbers

The `orderNo` field is your idempotency key. The gateway de-duplicates requests sharing the same `orderNo` within a configurable window. Rules:

- Generate `orderNo` on your server — never client-side.
- Use a globally unique identifier (UUID v4, or a prefixed timestamp+random combination).
- Store `orderNo` in your database **before** sending the API request so you can reconcile even if your server crashes after the gateway responds.
- Do not reuse `orderNo` for a new payment attempt; create a new order number.

> fa-solid fa-circle-info
>
> See the [Transaction Reconciliation](/docs/transaction-reconciliation) guide for the recommended database schema and reconciliation workflow.

Understand how the S2S Payment Gateway is structured, how your server communicates with the gateway, and how payment flows traverse the system.

## System Architecture

The S2S (Server-to-Server) Payment Gateway operates as a secure intermediary between your application server and the underlying payment networks (UPI, NPCI, card networks, bank portals).

```
┌─────────────────────────────────────────────────────────────────────┐
│                        MERCHANT ECOSYSTEM                           │
│                                                                     │
│   ┌──────────────┐      ┌──────────────┐     ┌──────────────────┐  │
│   │  Customer    │◄────►│  Merchant    │────►│  Merchant Server │  │
│   │  Browser /   │      │  Frontend    │     │  (Your Backend)  │  │
│   │  Mobile App  │      │  (Web/App)   │     └────────┬─────────┘  │
│   └──────────────┘      └──────────────┘              │            │
└───────────────────────────────────────────────────────┼────────────┘
                                                         │
                                              HTTPS + AES-256 + HMAC
                                                         │
┌───────────────────────────────────────────────────────▼────────────┐
│                     S2S PAYMENT GATEWAY                             │
│                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐  │
│  │  API Layer   │  │  Auth Layer  │  │  Encryption / Decryption │  │
│  │  (REST APIs) │  │  (HMAC-SHA256│  │  (AES-256-CBC)           │  │
│  └──────┬───────┘  └──────────────┘  └──────────────────────────┘  │
│         │                                                           │
│  ┌──────▼──────────────────────────────────────────────────────┐   │
│  │               PAYMENT ROUTING ENGINE                        │   │
│  │  UPI Handler │ NB Handler │ Card Handler │ Checkout Engine  │   │
│  └──────┬───────────┬────────────┬────────────────┬────────────┘   │
└─────────┼───────────┼────────────┼────────────────┼────────────────┘
          │           │            │                │
    ┌─────▼──┐  ┌─────▼──┐  ┌─────▼──┐      ┌─────▼──┐
    │  NPCI  │  │  Bank  │  │  Card  │      │Payment │
    │  UPI   │  │  Net   │  │Network │      │  Link  │
    │  Rails │  │Banking │  │(Visa/  │      │  Page  │
    └────────┘  └────────┘  │  MC)   │      └────────┘
                            └────────┘
```

<br />

## Request Lifecycle

Every API call follows this security pipeline:

1. **Payload construction** — Your server builds the JSON request body.
2. **AES-256 encryption** — The JSON body is encrypted using AES-256-CBC with your merchant-specific key and IV.
3. **Checksum generation** — An HMAC-SHA256 hash is computed from the pipe-delimited formula for the specific API.
4. **HTTP request** — The encrypted payload is sent over HTTPS with `X-Merchant-Id` and `X-Checksum` headers.
5. **Gateway validation** — The gateway decrypts the payload, validates the checksum, and processes the request.
6. **Encrypted response** — The gateway returns an AES-256-encrypted response body.
7. **Response decryption** — Your server decrypts the response using the same key and IV.
8. **Status evaluation** — Your application reads `response.status` and `response.statusCode` to determine the outcome.

<br />

## Core Components

| Component                        | Responsibility                                                           |
| -------------------------------- | ------------------------------------------------------------------------ |
| **VAS API** (`/vas/*`)           | Value-Added Services: VPA verification, instrument listing, bank listing |
| **Payment API** (`/payment/*`)   | Transaction initiation across all payment modes                          |
| **Merchant API** (`/merchant/*`) | Transaction status queries                                               |
| **Checkout API** (`/checkout/*`) | Hosted checkout sessions and payment link generation                     |
| **Webhook Engine**               | Asynchronous payment status notifications to your server                 |

<br />

## Security Architecture

The gateway enforces three independent security controls on every request:

<Columns layout="auto">
<Column>

**Transport Security**

* All communication over HTTPS/TLS 1.2+
* Certificate pinning recommended for mobile SDKs
* No plaintext HTTP allowed in production

**Authentication**

* HMAC-SHA256 checksum per request
* Formula varies by API endpoint
* `salt` is a shared secret — never client-side

</Column>
<Column>

**Payload Encryption**

* AES-256-CBC on request bodies
* AES-256-CBC on response bodies
* Unique key + IV pair per merchant

**Request Integrity**

* Checksum covers key transaction parameters
* Prevents parameter tampering in transit
* Gateway rejects any request with checksum mismatch

</Column>
</Columns>

<br />

## Data Flow by Payment Mode

| Payment Mode        | Customer Interaction         | Redirect Required  |
| ------------------- | ---------------------------- | ------------------ |
| UPI Collect         | Customer approves in UPI app | No                 |
| UPI Intent          | Deep-link to UPI app         | App switch         |
| Dynamic QR          | Customer scans QR code       | No                 |
| Net Banking         | Customer logs in at bank     | Yes (bank portal)  |
| Credit / Debit Card | Customer enters card details | Optional (3DS)     |
| Hosted Checkout     | Gateway-hosted page          | Yes (gateway page) |
| Payment Link        | Customer opens link          | Yes (gateway page) |

<br />

## Idempotency and Order Numbers

The `orderNo` field is your idempotency key. The gateway de-duplicates requests sharing the same `orderNo` within a configurable window. Rules:

- Generate `orderNo` on your server — never client-side.
- Use a globally unique identifier (UUID v4, or a prefixed timestamp+random combination).
- Store `orderNo` in your database **before** sending the API request so you can reconcile even if your server crashes after the gateway responds.
- Do not reuse `orderNo` for a new payment attempt; create a new order number.

> fa-solid fa-circle-info
>
> See the [Transaction Reconciliation](/docs/transaction-reconciliation) guide for the recommended database schema and reconciliation workflow.

Understand how the S2S Payment Gateway is structured, how your server communicates with the gateway, and how payment flows traverse the system.

## System Architecture

The S2S (Server-to-Server) Payment Gateway operates as a secure intermediary between your application server and the underlying payment networks (UPI, NPCI, card networks, bank portals).

```
┌─────────────────────────────────────────────────────────────────────┐
│                        MERCHANT ECOSYSTEM                           │
│                                                                     │
│   ┌──────────────┐      ┌──────────────┐     ┌──────────────────┐  │
│   │  Customer    │◄────►│  Merchant    │────►│  Merchant Server │  │
│   │  Browser /   │      │  Frontend    │     │  (Your Backend)  │  │
│   │  Mobile App  │      │  (Web/App)   │     └────────┬─────────┘  │
│   └──────────────┘      └──────────────┘              │            │
└───────────────────────────────────────────────────────┼────────────┘
                                                         │
                                              HTTPS + AES-256 + HMAC
                                                         │
┌───────────────────────────────────────────────────────▼────────────┐
│                     S2S PAYMENT GATEWAY                             │
│                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐  │
│  │  API Layer   │  │  Auth Layer  │  │  Encryption / Decryption │  │
│  │  (REST APIs) │  │  (HMAC-SHA256│  │  (AES-256-CBC)           │  │
│  └──────┬───────┘  └──────────────┘  └──────────────────────────┘  │
│         │                                                           │
│  ┌──────▼──────────────────────────────────────────────────────┐   │
│  │               PAYMENT ROUTING ENGINE                        │   │
│  │  UPI Handler │ NB Handler │ Card Handler │ Checkout Engine  │   │
│  └──────┬───────────┬────────────┬────────────────┬────────────┘   │
└─────────┼───────────┼────────────┼────────────────┼────────────────┘
          │           │            │                │
    ┌─────▼──┐  ┌─────▼──┐  ┌─────▼──┐      ┌─────▼──┐
    │  NPCI  │  │  Bank  │  │  Card  │      │Payment │
    │  UPI   │  │  Net   │  │Network │      │  Link  │
    │  Rails │  │Banking │  │(Visa/  │      │  Page  │
    └────────┘  └────────┘  │  MC)   │      └────────┘
                            └────────┘
```

<br />

## Request Lifecycle

Every API call follows this security pipeline:

1. **Payload construction** — Your server builds the JSON request body.
2. **AES-256 encryption** — The JSON body is encrypted using AES-256-CBC with your merchant-specific key and IV.
3. **Checksum generation** — An HMAC-SHA256 hash is computed from the pipe-delimited formula for the specific API.
4. **HTTP request** — The encrypted payload is sent over HTTPS with `X-Merchant-Id` and `X-Checksum` headers.
5. **Gateway validation** — The gateway decrypts the payload, validates the checksum, and processes the request.
6. **Encrypted response** — The gateway returns an AES-256-encrypted response body.
7. **Response decryption** — Your server decrypts the response using the same key and IV.
8. **Status evaluation** — Your application reads `response.status` and `response.statusCode` to determine the outcome.

<br />

## Core Components

| Component                        | Responsibility                                                           |
| -------------------------------- | ------------------------------------------------------------------------ |
| **VAS API** (`/vas/*`)           | Value-Added Services: VPA verification, instrument listing, bank listing |
| **Payment API** (`/payment/*`)   | Transaction initiation across all payment modes                          |
| **Merchant API** (`/merchant/*`) | Transaction status queries                                               |
| **Checkout API** (`/checkout/*`) | Hosted checkout sessions and payment link generation                     |
| **Webhook Engine**               | Asynchronous payment status notifications to your server                 |

<br />

## Security Architecture

The gateway enforces three independent security controls on every request:

<Columns layout="auto">
<Column>

**Transport Security**

* All communication over HTTPS/TLS 1.2+
* Certificate pinning recommended for mobile SDKs
* No plaintext HTTP allowed in production

**Authentication**

* HMAC-SHA256 checksum per request
* Formula varies by API endpoint
* `salt` is a shared secret — never client-side

</Column>
<Column>

**Payload Encryption**

* AES-256-CBC on request bodies
* AES-256-CBC on response bodies
* Unique key + IV pair per merchant

**Request Integrity**

* Checksum covers key transaction parameters
* Prevents parameter tampering in transit
* Gateway rejects any request with checksum mismatch

</Column>
</Columns>

<br />

## Data Flow by Payment Mode

| Payment Mode        | Customer Interaction         | Redirect Required  |
| ------------------- | ---------------------------- | ------------------ |
| UPI Collect         | Customer approves in UPI app | No                 |
| UPI Intent          | Deep-link to UPI app         | App switch         |
| Dynamic QR          | Customer scans QR code       | No                 |
| Net Banking         | Customer logs in at bank     | Yes (bank portal)  |
| Credit / Debit Card | Customer enters card details | Optional (3DS)     |
| Hosted Checkout     | Gateway-hosted page          | Yes (gateway page) |
| Payment Link        | Customer opens link          | Yes (gateway page) |

<br />

## Idempotency and Order Numbers

The `orderNo` field is your idempotency key. The gateway de-duplicates requests sharing the same `orderNo` within a configurable window. Rules:

- Generate `orderNo` on your server — never client-side.
- Use a globally unique identifier (UUID v4, or a prefixed timestamp+random combination).
- Store `orderNo` in your database **before** sending the API request so you can reconcile even if your server crashes after the gateway responds.
- Do not reuse `orderNo` for a new payment attempt; create a new order number.

> fa-solid fa-circle-info
>
> See the [Transaction Reconciliation](/docs/transaction-reconciliation) guide for the recommended database schema and reconciliation workflow.

Understand how the S2S Payment Gateway is structured, how your server communicates with the gateway, and how payment flows traverse the system.

## System Architecture

The S2S (Server-to-Server) Payment Gateway operates as a secure intermediary between your application server and the underlying payment networks (UPI, NPCI, card networks, bank portals).

```
┌─────────────────────────────────────────────────────────────────────┐
│                        MERCHANT ECOSYSTEM                           │
│                                                                     │
│   ┌──────────────┐      ┌──────────────┐     ┌──────────────────┐  │
│   │  Customer    │◄────►│  Merchant    │────►│  Merchant Server │  │
│   │  Browser /   │      │  Frontend    │     │  (Your Backend)  │  │
│   │  Mobile App  │      │  (Web/App)   │     └────────┬─────────┘  │
│   └──────────────┘      └──────────────┘              │            │
└───────────────────────────────────────────────────────┼────────────┘
                                                         │
                                              HTTPS + AES-256 + HMAC
                                                         │
┌───────────────────────────────────────────────────────▼────────────┐
│                     S2S PAYMENT GATEWAY                             │
│                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐  │
│  │  API Layer   │  │  Auth Layer  │  │  Encryption / Decryption │  │
│  │  (REST APIs) │  │  (HMAC-SHA256│  │  (AES-256-CBC)           │  │
│  └──────┬───────┘  └──────────────┘  └──────────────────────────┘  │
│         │                                                           │
│  ┌──────▼──────────────────────────────────────────────────────┐   │
│  │               PAYMENT ROUTING ENGINE                        │   │
│  │  UPI Handler │ NB Handler │ Card Handler │ Checkout Engine  │   │
│  └──────┬───────────┬────────────┬────────────────┬────────────┘   │
└─────────┼───────────┼────────────┼────────────────┼────────────────┘
          │           │            │                │
    ┌─────▼──┐  ┌─────▼──┐  ┌─────▼──┐      ┌─────▼──┐
    │  NPCI  │  │  Bank  │  │  Card  │      │Payment │
    │  UPI   │  │  Net   │  │Network │      │  Link  │
    │  Rails │  │Banking │  │(Visa/  │      │  Page  │
    └────────┘  └────────┘  │  MC)   │      └────────┘
                            └────────┘
```

<br />

## Request Lifecycle

Every API call follows this security pipeline:

1. **Payload construction** — Your server builds the JSON request body.
2. **AES-256 encryption** — The JSON body is encrypted using AES-256-CBC with your merchant-specific key and IV.
3. **Checksum generation** — An HMAC-SHA256 hash is computed from the pipe-delimited formula for the specific API.
4. **HTTP request** — The encrypted payload is sent over HTTPS with `X-Merchant-Id` and `X-Checksum` headers.
5. **Gateway validation** — The gateway decrypts the payload, validates the checksum, and processes the request.
6. **Encrypted response** — The gateway returns an AES-256-encrypted response body.
7. **Response decryption** — Your server decrypts the response using the same key and IV.
8. **Status evaluation** — Your application reads `response.status` and `response.statusCode` to determine the outcome.

<br />

## Core Components

| Component                        | Responsibility                                                           |
| -------------------------------- | ------------------------------------------------------------------------ |
| **VAS API** (`/vas/*`)           | Value-Added Services: VPA verification, instrument listing, bank listing |
| **Payment API** (`/payment/*`)   | Transaction initiation across all payment modes                          |
| **Merchant API** (`/merchant/*`) | Transaction status queries                                               |
| **Checkout API** (`/checkout/*`) | Hosted checkout sessions and payment link generation                     |
| **Webhook Engine**               | Asynchronous payment status notifications to your server                 |

<br />

## Security Architecture

The gateway enforces three independent security controls on every request:

<Columns layout="auto">
<Column>

**Transport Security**

* All communication over HTTPS/TLS 1.2+
* Certificate pinning recommended for mobile SDKs
* No plaintext HTTP allowed in production

**Authentication**

* HMAC-SHA256 checksum per request
* Formula varies by API endpoint
* `salt` is a shared secret — never client-side

</Column>
<Column>

**Payload Encryption**

* AES-256-CBC on request bodies
* AES-256-CBC on response bodies
* Unique key + IV pair per merchant

**Request Integrity**

* Checksum covers key transaction parameters
* Prevents parameter tampering in transit
* Gateway rejects any request with checksum mismatch

</Column>
</Columns>

<br />

## Data Flow by Payment Mode

| Payment Mode        | Customer Interaction         | Redirect Required  |
| ------------------- | ---------------------------- | ------------------ |
| UPI Collect         | Customer approves in UPI app | No                 |
| UPI Intent          | Deep-link to UPI app         | App switch         |
| Dynamic QR          | Customer scans QR code       | No                 |
| Net Banking         | Customer logs in at bank     | Yes (bank portal)  |
| Credit / Debit Card | Customer enters card details | Optional (3DS)     |
| Hosted Checkout     | Gateway-hosted page          | Yes (gateway page) |
| Payment Link        | Customer opens link          | Yes (gateway page) |

<br />

## Idempotency and Order Numbers

The `orderNo` field is your idempotency key. The gateway de-duplicates requests sharing the same `orderNo` within a configurable window. Rules:

- Generate `orderNo` on your server — never client-side.
- Use a globally unique identifier (UUID v4, or a prefixed timestamp+random combination).
- Store `orderNo` in your database **before** sending the API request so you can reconcile even if your server crashes after the gateway responds.
- Do not reuse `orderNo` for a new payment attempt; create a new order number.

> fa-solid fa-circle-info
>
> See the [Transaction Reconciliation](/docs/transaction-reconciliation) guide for the recommended database schema and reconciliation workflow.

<br />
