---
title: '# Overview'
excerpt: >-
  S2S Payment Gateway — production-grade REST API platform supporting UPI, Net
  Banking, Credit Card, and Debit Card payments with AES-256 encryption and
  HMAC-SHA256 authentication.  Use the S2S Payment Gateway REST APIs to accept
  payments across UPI, Net Banking, Credit Cards, and Debit Cards — with
  end-to-end AES-256 encryption and HMAC-SHA256 request authentication.  ##
deprecated: false
hidden: false
metadata:
  robots: index
---
## Platform Highlights

<Cards columns={3}>
  <Card title="HMAC-SHA256 Auth" icon="fa-duotone fa-shield-halved">
    Every request is authenticated with a merchant-specific checksum derived from request parameters and a shared salt.
  </Card>
  <Card title="AES-256 Encryption" icon="fa-duotone fa-lock">
    Request payloads are encrypted before transmission. Responses are returned encrypted and must be decrypted client-side.
  </Card>
  <Card title="Multi-Mode Payments" icon="fa-duotone fa-credit-card">
    Accept UPI Collect, Dynamic QR, UPI Intent, Net Banking, Credit Card, and Debit Card — all through a unified API.
  </Card>
  <Card title="Hosted Checkout" icon="fa-duotone fa-cart-shopping">
    Redirect customers to a gateway-hosted checkout page — no PCI-DSS scope on your servers.
  </Card>
  <Card title="Payment Links" icon="fa-duotone fa-link">
    Generate shareable payment links with configurable expiry — no integration required on the customer side.
  </Card>
  <Card title="Webhook Notifications" icon="fa-duotone fa-bell">
    Receive real-time payment status notifications via signed webhooks with retry logic.
  </Card>
</Cards>

<br />

## Environment URLs

| Environment   | Base URL                         |
| ------------- | -------------------------------- |
| UAT (Sandbox) | `https://<uat-host>:8085/s2s/v1` |
| Production    | `https://<prod-host>/s2s/v1`     |

> fa-solid fa-triangle-exclamation
>
> Never use production credentials in the UAT environment, or UAT credentials against production endpoints. Maintain separate key pairs for each environment.

<br />

## Supported Payment Methods

| Method      | `paymentMode` | `paymentCode`           | Description                                 |
| ----------- | ------------- | ----------------------- | ------------------------------------------- |
| UPI Collect | `UPI`         | `UPII`                  | Push collect request to customer's UPI app  |
| UPI Intent  | `UPI`         | `UPIC`                  | Deep-link to UPI app on the same device     |
| Dynamic QR  | `UPI`         | `DQR`                   | Generate scannable QR for UPI payment       |
| Net Banking | `NB`          | Bank code (e.g. `HDFC`) | Redirect to bank's net banking portal       |
| Credit Card | `CC`          | `CC`                    | Accept Visa, Mastercard, RuPay credit cards |
| Debit Card  | `DC`          | `DC`                    | Accept Visa, Mastercard, RuPay debit cards  |

<br />

## API Reference

<Cards>
  <Card title="Verify VPA" href="/reference/verify-vpa" icon="fa-duotone fa-badge-check">
    Validate a UPI VPA before initiating a UPI Collect payment.
  </Card>
  <Card title="Netbanking List" href="/reference/netbanking-list" icon="fa-duotone fa-building-columns">
    Retrieve the list of available net banking options.
  </Card>
  <Card title="Enabled Instruments" href="/reference/enabled-instruments" icon="fa-duotone fa-credit-card">
    List all payment instruments enabled for your merchant account.
  </Card>
  <Card title="Initiate Transaction" href="/reference/initiate-transaction" icon="fa-duotone fa-money-check-dollar">
    Create a new payment transaction across any supported payment mode.
  </Card>
  <Card title="Transaction Status" href="/reference/transaction-status" icon="fa-duotone fa-list-check">
    Query the current status of any transaction by RRN or transaction ID.
  </Card>
  <Card title="Checkout Intent" href="/reference/checkout-intent" icon="fa-duotone fa-cart-shopping">
    Create a hosted checkout session and redirect the customer.
  </Card>
  <Card title="Checkout Payment Link" href="/reference/checkout-payment-link" icon="fa-duotone fa-link">
    Generate a shareable payment link with configurable expiry.
  </Card>
</Cards>

<br />

## Quick Navigation

<Cards columns={2}>
  <Card title="Quick Start Guide" href="/docs/quickstart" icon="fa-duotone fa-rocket">
    Get your first test transaction running in under 10 minutes.
  </Card>
  <Card title="Authentication" href="/docs/authentication" icon="fa-duotone fa-key">
    Understand HMAC-SHA256 checksum generation with code samples in 5 languages.
  </Card>
  <Card title="Encryption Guide" href="/docs/encryption" icon="fa-duotone fa-lock">
    Encrypt requests and decrypt responses using AES-256.
  </Card>
  <Card title="API Flows" href="/docs/api-flows" icon="fa-duotone fa-diagram-project">
    End-to-end flow diagrams for every payment method.
  </Card>
  <Card title="Error Handling" href="/docs/error-handling" icon="fa-duotone fa-circle-exclamation">
    HTTP status codes, error payloads, and resolution steps.
  </Card>
  <Card title="Webhooks" href="/docs/webhooks" icon="fa-duotone fa-webhook">
    Configure and verify real-time payment notifications.
  </Card>
  <Card title="Best Practices" href="/docs/security-best-practices" icon="fa-duotone fa-shield-check">
    Security, retry strategy, idempotency, and reconciliation.
  </Card>
  <Card title="Go-Live Checklist" href="/docs/production-checklist" icon="fa-duotone fa-clipboard-check">
    Everything you need to verify before going live in production.
  </Card>
</Cards>

<br />

## Checksum Formulas Reference

| API                  | Checksum Formula                                               |
| -------------------- | -------------------------------------------------------------- |
| Verify VPA           | `mid\|VPA_VERIFY\|vpa\|salt`                                   |
| Netbanking List      | `mid\|NETBANKING_LIST\|salt`                                   |
| Enabled Instruments  | `mid\|ENABLED_INSTRUMENTS\|salt`                               |
| Initiate Transaction | `mid\|orderNo\|rrn\|txnAmount\|paymentMode\|paymentCode\|salt` |
| Transaction Status   | `mid\|rrn\|txnId\|salt`                                        |

> fa-solid fa-circle-info
>
> The `salt` value is a secret shared between your server and the gateway. Never expose it in client-side code, mobile apps, or browser JavaScript.

Use the S2S Payment Gateway REST APIs to accept payments across UPI, Net Banking, Credit Cards, and Debit Cards — with end-to-end AES-256 encryption and HMAC-SHA256 request authentication.

## Platform Highlights

<Cards columns={3}>
  <Card title="HMAC-SHA256 Auth" icon="fa-duotone fa-shield-halved">
    Every request is authenticated with a merchant-specific checksum derived from request parameters and a shared salt.
  </Card>
  <Card title="AES-256 Encryption" icon="fa-duotone fa-lock">
    Request payloads are encrypted before transmission. Responses are returned encrypted and must be decrypted client-side.
  </Card>
  <Card title="Multi-Mode Payments" icon="fa-duotone fa-credit-card">
    Accept UPI Collect, Dynamic QR, UPI Intent, Net Banking, Credit Card, and Debit Card — all through a unified API.
  </Card>
  <Card title="Hosted Checkout" icon="fa-duotone fa-cart-shopping">
    Redirect customers to a gateway-hosted checkout page — no PCI-DSS scope on your servers.
  </Card>
  <Card title="Payment Links" icon="fa-duotone fa-link">
    Generate shareable payment links with configurable expiry — no integration required on the customer side.
  </Card>
  <Card title="Webhook Notifications" icon="fa-duotone fa-bell">
    Receive real-time payment status notifications via signed webhooks with retry logic.
  </Card>
</Cards>

<br />

## Environment URLs

| Environment   | Base URL                         |
| ------------- | -------------------------------- |
| UAT (Sandbox) | `https://<uat-host>:8085/s2s/v1` |
| Production    | `https://<prod-host>/s2s/v1`     |

> fa-solid fa-triangle-exclamation
>
> Never use production credentials in the UAT environment, or UAT credentials against production endpoints. Maintain separate key pairs for each environment.

<br />

## Supported Payment Methods

| Method      | `paymentMode` | `paymentCode`           | Description                                 |
| ----------- | ------------- | ----------------------- | ------------------------------------------- |
| UPI Collect | `UPI`         | `UPII`                  | Push collect request to customer's UPI app  |
| UPI Intent  | `UPI`         | `UPIC`                  | Deep-link to UPI app on the same device     |
| Dynamic QR  | `UPI`         | `DQR`                   | Generate scannable QR for UPI payment       |
| Net Banking | `NB`          | Bank code (e.g. `HDFC`) | Redirect to bank's net banking portal       |
| Credit Card | `CC`          | `CC`                    | Accept Visa, Mastercard, RuPay credit cards |
| Debit Card  | `DC`          | `DC`                    | Accept Visa, Mastercard, RuPay debit cards  |

<br />

## API Reference

<Cards>
  <Card title="Verify VPA" href="/reference/verify-vpa" icon="fa-duotone fa-badge-check">
    Validate a UPI VPA before initiating a UPI Collect payment.
  </Card>
  <Card title="Netbanking List" href="/reference/netbanking-list" icon="fa-duotone fa-building-columns">
    Retrieve the list of available net banking options.
  </Card>
  <Card title="Enabled Instruments" href="/reference/enabled-instruments" icon="fa-duotone fa-credit-card">
    List all payment instruments enabled for your merchant account.
  </Card>
  <Card title="Initiate Transaction" href="/reference/initiate-transaction" icon="fa-duotone fa-money-check-dollar">
    Create a new payment transaction across any supported payment mode.
  </Card>
  <Card title="Transaction Status" href="/reference/transaction-status" icon="fa-duotone fa-list-check">
    Query the current status of any transaction by RRN or transaction ID.
  </Card>
  <Card title="Checkout Intent" href="/reference/checkout-intent" icon="fa-duotone fa-cart-shopping">
    Create a hosted checkout session and redirect the customer.
  </Card>
  <Card title="Checkout Payment Link" href="/reference/checkout-payment-link" icon="fa-duotone fa-link">
    Generate a shareable payment link with configurable expiry.
  </Card>
</Cards>

<br />

## Quick Navigation

<Cards columns={2}>
  <Card title="Quick Start Guide" href="/docs/quickstart" icon="fa-duotone fa-rocket">
    Get your first test transaction running in under 10 minutes.
  </Card>
  <Card title="Authentication" href="/docs/authentication" icon="fa-duotone fa-key">
    Understand HMAC-SHA256 checksum generation with code samples in 5 languages.
  </Card>
  <Card title="Encryption Guide" href="/docs/encryption" icon="fa-duotone fa-lock">
    Encrypt requests and decrypt responses using AES-256.
  </Card>
  <Card title="API Flows" href="/docs/api-flows" icon="fa-duotone fa-diagram-project">
    End-to-end flow diagrams for every payment method.
  </Card>
  <Card title="Error Handling" href="/docs/error-handling" icon="fa-duotone fa-circle-exclamation">
    HTTP status codes, error payloads, and resolution steps.
  </Card>
  <Card title="Webhooks" href="/docs/webhooks" icon="fa-duotone fa-webhook">
    Configure and verify real-time payment notifications.
  </Card>
  <Card title="Best Practices" href="/docs/security-best-practices" icon="fa-duotone fa-shield-check">
    Security, retry strategy, idempotency, and reconciliation.
  </Card>
  <Card title="Go-Live Checklist" href="/docs/production-checklist" icon="fa-duotone fa-clipboard-check">
    Everything you need to verify before going live in production.
  </Card>
</Cards>

<br />

## Checksum Formulas Reference

| API                  | Checksum Formula                                               |
| -------------------- | -------------------------------------------------------------- |
| Verify VPA           | `mid\|VPA_VERIFY\|vpa\|salt`                                   |
| Netbanking List      | `mid\|NETBANKING_LIST\|salt`                                   |
| Enabled Instruments  | `mid\|ENABLED_INSTRUMENTS\|salt`                               |
| Initiate Transaction | `mid\|orderNo\|rrn\|txnAmount\|paymentMode\|paymentCode\|salt` |
| Transaction Status   | `mid\|rrn\|txnId\|salt`                                        |

> fa-solid fa-circle-info
>
> The `salt` value is a secret shared between your server and the gateway. Never expose it in client-side code, mobile apps, or browser JavaScript.

Use the S2S Payment Gateway REST APIs to accept payments across UPI, Net Banking, Credit Cards, and Debit Cards — with end-to-end AES-256 encryption and HMAC-SHA256 request authentication.

## Platform Highlights

<Cards columns={3}>
  <Card title="HMAC-SHA256 Auth" icon="fa-duotone fa-shield-halved">
    Every request is authenticated with a merchant-specific checksum derived from request parameters and a shared salt.
  </Card>
  <Card title="AES-256 Encryption" icon="fa-duotone fa-lock">
    Request payloads are encrypted before transmission. Responses are returned encrypted and must be decrypted client-side.
  </Card>
  <Card title="Multi-Mode Payments" icon="fa-duotone fa-credit-card">
    Accept UPI Collect, Dynamic QR, UPI Intent, Net Banking, Credit Card, and Debit Card — all through a unified API.
  </Card>
  <Card title="Hosted Checkout" icon="fa-duotone fa-cart-shopping">
    Redirect customers to a gateway-hosted checkout page — no PCI-DSS scope on your servers.
  </Card>
  <Card title="Payment Links" icon="fa-duotone fa-link">
    Generate shareable payment links with configurable expiry — no integration required on the customer side.
  </Card>
  <Card title="Webhook Notifications" icon="fa-duotone fa-bell">
    Receive real-time payment status notifications via signed webhooks with retry logic.
  </Card>
</Cards>

<br />

## Environment URLs

| Environment   | Base URL                         |
| ------------- | -------------------------------- |
| UAT (Sandbox) | `https://<uat-host>:8085/s2s/v1` |
| Production    | `https://<prod-host>/s2s/v1`     |

> fa-solid fa-triangle-exclamation
>
> Never use production credentials in the UAT environment, or UAT credentials against production endpoints. Maintain separate key pairs for each environment.

<br />

## Supported Payment Methods

| Method      | `paymentMode` | `paymentCode`           | Description                                 |
| ----------- | ------------- | ----------------------- | ------------------------------------------- |
| UPI Collect | `UPI`         | `UPII`                  | Push collect request to customer's UPI app  |
| UPI Intent  | `UPI`         | `UPIC`                  | Deep-link to UPI app on the same device     |
| Dynamic QR  | `UPI`         | `DQR`                   | Generate scannable QR for UPI payment       |
| Net Banking | `NB`          | Bank code (e.g. `HDFC`) | Redirect to bank's net banking portal       |
| Credit Card | `CC`          | `CC`                    | Accept Visa, Mastercard, RuPay credit cards |
| Debit Card  | `DC`          | `DC`                    | Accept Visa, Mastercard, RuPay debit cards  |

<br />

## API Reference

<Cards>
  <Card title="Verify VPA" href="/reference/verify-vpa" icon="fa-duotone fa-badge-check">
    Validate a UPI VPA before initiating a UPI Collect payment.
  </Card>
  <Card title="Netbanking List" href="/reference/netbanking-list" icon="fa-duotone fa-building-columns">
    Retrieve the list of available net banking options.
  </Card>
  <Card title="Enabled Instruments" href="/reference/enabled-instruments" icon="fa-duotone fa-credit-card">
    List all payment instruments enabled for your merchant account.
  </Card>
  <Card title="Initiate Transaction" href="/reference/initiate-transaction" icon="fa-duotone fa-money-check-dollar">
    Create a new payment transaction across any supported payment mode.
  </Card>
  <Card title="Transaction Status" href="/reference/transaction-status" icon="fa-duotone fa-list-check">
    Query the current status of any transaction by RRN or transaction ID.
  </Card>
  <Card title="Checkout Intent" href="/reference/checkout-intent" icon="fa-duotone fa-cart-shopping">
    Create a hosted checkout session and redirect the customer.
  </Card>
  <Card title="Checkout Payment Link" href="/reference/checkout-payment-link" icon="fa-duotone fa-link">
    Generate a shareable payment link with configurable expiry.
  </Card>
</Cards>

<br />

## Quick Navigation

<Cards columns={2}>
  <Card title="Quick Start Guide" href="/docs/quickstart" icon="fa-duotone fa-rocket">
    Get your first test transaction running in under 10 minutes.
  </Card>
  <Card title="Authentication" href="/docs/authentication" icon="fa-duotone fa-key">
    Understand HMAC-SHA256 checksum generation with code samples in 5 languages.
  </Card>
  <Card title="Encryption Guide" href="/docs/encryption" icon="fa-duotone fa-lock">
    Encrypt requests and decrypt responses using AES-256.
  </Card>
  <Card title="API Flows" href="/docs/api-flows" icon="fa-duotone fa-diagram-project">
    End-to-end flow diagrams for every payment method.
  </Card>
  <Card title="Error Handling" href="/docs/error-handling" icon="fa-duotone fa-circle-exclamation">
    HTTP status codes, error payloads, and resolution steps.
  </Card>
  <Card title="Webhooks" href="/docs/webhooks" icon="fa-duotone fa-webhook">
    Configure and verify real-time payment notifications.
  </Card>
  <Card title="Best Practices" href="/docs/security-best-practices" icon="fa-duotone fa-shield-check">
    Security, retry strategy, idempotency, and reconciliation.
  </Card>
  <Card title="Go-Live Checklist" href="/docs/production-checklist" icon="fa-duotone fa-clipboard-check">
    Everything you need to verify before going live in production.
  </Card>
</Cards>

<br />

## Checksum Formulas Reference

| API                  | Checksum Formula                                               |
| -------------------- | -------------------------------------------------------------- |
| Verify VPA           | `mid\|VPA_VERIFY\|vpa\|salt`                                   |
| Netbanking List      | `mid\|NETBANKING_LIST\|salt`                                   |
| Enabled Instruments  | `mid\|ENABLED_INSTRUMENTS\|salt`                               |
| Initiate Transaction | `mid\|orderNo\|rrn\|txnAmount\|paymentMode\|paymentCode\|salt` |
| Transaction Status   | `mid\|rrn\|txnId\|salt`                                        |

> fa-solid fa-circle-info
>
> The `salt` value is a secret shared between your server and the gateway. Never expose it in client-side code, mobile apps, or browser JavaScript.

Use the S2S Payment Gateway REST APIs to accept payments across UPI, Net Banking, Credit Cards, and Debit Cards — with end-to-end AES-256 encryption and HMAC-SHA256 request authentication.

## Platform Highlights

<Cards columns={3}>
  <Card title="HMAC-SHA256 Auth" icon="fa-duotone fa-shield-halved">
    Every request is authenticated with a merchant-specific checksum derived from request parameters and a shared salt.
  </Card>
  <Card title="AES-256 Encryption" icon="fa-duotone fa-lock">
    Request payloads are encrypted before transmission. Responses are returned encrypted and must be decrypted client-side.
  </Card>
  <Card title="Multi-Mode Payments" icon="fa-duotone fa-credit-card">
    Accept UPI Collect, Dynamic QR, UPI Intent, Net Banking, Credit Card, and Debit Card — all through a unified API.
  </Card>
  <Card title="Hosted Checkout" icon="fa-duotone fa-cart-shopping">
    Redirect customers to a gateway-hosted checkout page — no PCI-DSS scope on your servers.
  </Card>
  <Card title="Payment Links" icon="fa-duotone fa-link">
    Generate shareable payment links with configurable expiry — no integration required on the customer side.
  </Card>
  <Card title="Webhook Notifications" icon="fa-duotone fa-bell">
    Receive real-time payment status notifications via signed webhooks with retry logic.
  </Card>
</Cards>

<br />

## Environment URLs

| Environment   | Base URL                         |
| ------------- | -------------------------------- |
| UAT (Sandbox) | `https://<uat-host>:8085/s2s/v1` |
| Production    | `https://<prod-host>/s2s/v1`     |

> fa-solid fa-triangle-exclamation
>
> Never use production credentials in the UAT environment, or UAT credentials against production endpoints. Maintain separate key pairs for each environment.

<br />

## Supported Payment Methods

| Method      | `paymentMode` | `paymentCode`           | Description                                 |
| ----------- | ------------- | ----------------------- | ------------------------------------------- |
| UPI Collect | `UPI`         | `UPII`                  | Push collect request to customer's UPI app  |
| UPI Intent  | `UPI`         | `UPIC`                  | Deep-link to UPI app on the same device     |
| Dynamic QR  | `UPI`         | `DQR`                   | Generate scannable QR for UPI payment       |
| Net Banking | `NB`          | Bank code (e.g. `HDFC`) | Redirect to bank's net banking portal       |
| Credit Card | `CC`          | `CC`                    | Accept Visa, Mastercard, RuPay credit cards |
| Debit Card  | `DC`          | `DC`                    | Accept Visa, Mastercard, RuPay debit cards  |

<br />

## API Reference

<Cards>
  <Card title="Verify VPA" href="/reference/verify-vpa" icon="fa-duotone fa-badge-check">
    Validate a UPI VPA before initiating a UPI Collect payment.
  </Card>
  <Card title="Netbanking List" href="/reference/netbanking-list" icon="fa-duotone fa-building-columns">
    Retrieve the list of available net banking options.
  </Card>
  <Card title="Enabled Instruments" href="/reference/enabled-instruments" icon="fa-duotone fa-credit-card">
    List all payment instruments enabled for your merchant account.
  </Card>
  <Card title="Initiate Transaction" href="/reference/initiate-transaction" icon="fa-duotone fa-money-check-dollar">
    Create a new payment transaction across any supported payment mode.
  </Card>
  <Card title="Transaction Status" href="/reference/transaction-status" icon="fa-duotone fa-list-check">
    Query the current status of any transaction by RRN or transaction ID.
  </Card>
  <Card title="Checkout Intent" href="/reference/checkout-intent" icon="fa-duotone fa-cart-shopping">
    Create a hosted checkout session and redirect the customer.
  </Card>
  <Card title="Checkout Payment Link" href="/reference/checkout-payment-link" icon="fa-duotone fa-link">
    Generate a shareable payment link with configurable expiry.
  </Card>
</Cards>

<br />

## Quick Navigation

<Cards columns={2}>
  <Card title="Quick Start Guide" href="/docs/quickstart" icon="fa-duotone fa-rocket">
    Get your first test transaction running in under 10 minutes.
  </Card>
  <Card title="Authentication" href="/docs/authentication" icon="fa-duotone fa-key">
    Understand HMAC-SHA256 checksum generation with code samples in 5 languages.
  </Card>
  <Card title="Encryption Guide" href="/docs/encryption" icon="fa-duotone fa-lock">
    Encrypt requests and decrypt responses using AES-256.
  </Card>
  <Card title="API Flows" href="/docs/api-flows" icon="fa-duotone fa-diagram-project">
    End-to-end flow diagrams for every payment method.
  </Card>
  <Card title="Error Handling" href="/docs/error-handling" icon="fa-duotone fa-circle-exclamation">
    HTTP status codes, error payloads, and resolution steps.
  </Card>
  <Card title="Webhooks" href="/docs/webhooks" icon="fa-duotone fa-webhook">
    Configure and verify real-time payment notifications.
  </Card>
  <Card title="Best Practices" href="/docs/security-best-practices" icon="fa-duotone fa-shield-check">
    Security, retry strategy, idempotency, and reconciliation.
  </Card>
  <Card title="Go-Live Checklist" href="/docs/production-checklist" icon="fa-duotone fa-clipboard-check">
    Everything you need to verify before going live in production.
  </Card>
</Cards>

<br />

## Checksum Formulas Reference

| API                  | Checksum Formula                                               |
| -------------------- | -------------------------------------------------------------- |
| Verify VPA           | `mid\|VPA_VERIFY\|vpa\|salt`                                   |
| Netbanking List      | `mid\|NETBANKING_LIST\|salt`                                   |
| Enabled Instruments  | `mid\|ENABLED_INSTRUMENTS\|salt`                               |
| Initiate Transaction | `mid\|orderNo\|rrn\|txnAmount\|paymentMode\|paymentCode\|salt` |
| Transaction Status   | `mid\|rrn\|txnId\|salt`                                        |

> fa-solid fa-circle-info
>
> The `salt` value is a secret shared between your server and the gateway. Never expose it in client-side code, mobile apps, or browser JavaScript.

Use the S2S Payment Gateway REST APIs to accept payments across UPI, Net Banking, Credit Cards, and Debit Cards — with end-to-end AES-256 encryption and HMAC-SHA256 request authentication.

## Platform Highlights

<Cards columns={3}>
  <Card title="HMAC-SHA256 Auth" icon="fa-duotone fa-shield-halved">
    Every request is authenticated with a merchant-specific checksum derived from request parameters and a shared salt.
  </Card>
  <Card title="AES-256 Encryption" icon="fa-duotone fa-lock">
    Request payloads are encrypted before transmission. Responses are returned encrypted and must be decrypted client-side.
  </Card>
  <Card title="Multi-Mode Payments" icon="fa-duotone fa-credit-card">
    Accept UPI Collect, Dynamic QR, UPI Intent, Net Banking, Credit Card, and Debit Card — all through a unified API.
  </Card>
  <Card title="Hosted Checkout" icon="fa-duotone fa-cart-shopping">
    Redirect customers to a gateway-hosted checkout page — no PCI-DSS scope on your servers.
  </Card>
  <Card title="Payment Links" icon="fa-duotone fa-link">
    Generate shareable payment links with configurable expiry — no integration required on the customer side.
  </Card>
  <Card title="Webhook Notifications" icon="fa-duotone fa-bell">
    Receive real-time payment status notifications via signed webhooks with retry logic.
  </Card>
</Cards>

<br />

## Environment URLs

| Environment   | Base URL                         |
| ------------- | -------------------------------- |
| UAT (Sandbox) | `https://<uat-host>:8085/s2s/v1` |
| Production    | `https://<prod-host>/s2s/v1`     |

> fa-solid fa-triangle-exclamation
>
> Never use production credentials in the UAT environment, or UAT credentials against production endpoints. Maintain separate key pairs for each environment.

<br />

## Supported Payment Methods

| Method      | `paymentMode` | `paymentCode`           | Description                                 |
| ----------- | ------------- | ----------------------- | ------------------------------------------- |
| UPI Collect | `UPI`         | `UPII`                  | Push collect request to customer's UPI app  |
| UPI Intent  | `UPI`         | `UPIC`                  | Deep-link to UPI app on the same device     |
| Dynamic QR  | `UPI`         | `DQR`                   | Generate scannable QR for UPI payment       |
| Net Banking | `NB`          | Bank code (e.g. `HDFC`) | Redirect to bank's net banking portal       |
| Credit Card | `CC`          | `CC`                    | Accept Visa, Mastercard, RuPay credit cards |
| Debit Card  | `DC`          | `DC`                    | Accept Visa, Mastercard, RuPay debit cards  |

<br />

## API Reference

<Cards>
  <Card title="Verify VPA" href="/reference/verify-vpa" icon="fa-duotone fa-badge-check">
    Validate a UPI VPA before initiating a UPI Collect payment.
  </Card>
  <Card title="Netbanking List" href="/reference/netbanking-list" icon="fa-duotone fa-building-columns">
    Retrieve the list of available net banking options.
  </Card>
  <Card title="Enabled Instruments" href="/reference/enabled-instruments" icon="fa-duotone fa-credit-card">
    List all payment instruments enabled for your merchant account.
  </Card>
  <Card title="Initiate Transaction" href="/reference/initiate-transaction" icon="fa-duotone fa-money-check-dollar">
    Create a new payment transaction across any supported payment mode.
  </Card>
  <Card title="Transaction Status" href="/reference/transaction-status" icon="fa-duotone fa-list-check">
    Query the current status of any transaction by RRN or transaction ID.
  </Card>
  <Card title="Checkout Intent" href="/reference/checkout-intent" icon="fa-duotone fa-cart-shopping">
    Create a hosted checkout session and redirect the customer.
  </Card>
  <Card title="Checkout Payment Link" href="/reference/checkout-payment-link" icon="fa-duotone fa-link">
    Generate a shareable payment link with configurable expiry.
  </Card>
</Cards>

<br />

## Quick Navigation

<Cards columns={2}>
  <Card title="Quick Start Guide" href="/docs/quickstart" icon="fa-duotone fa-rocket">
    Get your first test transaction running in under 10 minutes.
  </Card>
  <Card title="Authentication" href="/docs/authentication" icon="fa-duotone fa-key">
    Understand HMAC-SHA256 checksum generation with code samples in 5 languages.
  </Card>
  <Card title="Encryption Guide" href="/docs/encryption" icon="fa-duotone fa-lock">
    Encrypt requests and decrypt responses using AES-256.
  </Card>
  <Card title="API Flows" href="/docs/api-flows" icon="fa-duotone fa-diagram-project">
    End-to-end flow diagrams for every payment method.
  </Card>
  <Card title="Error Handling" href="/docs/error-handling" icon="fa-duotone fa-circle-exclamation">
    HTTP status codes, error payloads, and resolution steps.
  </Card>
  <Card title="Webhooks" href="/docs/webhooks" icon="fa-duotone fa-webhook">
    Configure and verify real-time payment notifications.
  </Card>
  <Card title="Best Practices" href="/docs/security-best-practices" icon="fa-duotone fa-shield-check">
    Security, retry strategy, idempotency, and reconciliation.
  </Card>
  <Card title="Go-Live Checklist" href="/docs/production-checklist" icon="fa-duotone fa-clipboard-check">
    Everything you need to verify before going live in production.
  </Card>
</Cards>

<br />

## Checksum Formulas Reference

| API                  | Checksum Formula                                               |
| -------------------- | -------------------------------------------------------------- |
| Verify VPA           | `mid\|VPA_VERIFY\|vpa\|salt`                                   |
| Netbanking List      | `mid\|NETBANKING_LIST\|salt`                                   |
| Enabled Instruments  | `mid\|ENABLED_INSTRUMENTS\|salt`                               |
| Initiate Transaction | `mid\|orderNo\|rrn\|txnAmount\|paymentMode\|paymentCode\|salt` |
| Transaction Status   | `mid\|rrn\|txnId\|salt`                                        |

> fa-solid fa-circle-info
>
> The `salt` value is a secret shared between your server and the gateway. Never expose it in client-side code, mobile apps, or browser JavaScript.

Use the S2S Payment Gateway REST APIs to accept payments across UPI, Net Banking, Credit Cards, and Debit Cards — with end-to-end AES-256 encryption and HMAC-SHA256 request authentication.

## Platform Highlights

<Cards columns={3}>
  <Card title="HMAC-SHA256 Auth" icon="fa-duotone fa-shield-halved">
    Every request is authenticated with a merchant-specific checksum derived from request parameters and a shared salt.
  </Card>
  <Card title="AES-256 Encryption" icon="fa-duotone fa-lock">
    Request payloads are encrypted before transmission. Responses are returned encrypted and must be decrypted client-side.
  </Card>
  <Card title="Multi-Mode Payments" icon="fa-duotone fa-credit-card">
    Accept UPI Collect, Dynamic QR, UPI Intent, Net Banking, Credit Card, and Debit Card — all through a unified API.
  </Card>
  <Card title="Hosted Checkout" icon="fa-duotone fa-cart-shopping">
    Redirect customers to a gateway-hosted checkout page — no PCI-DSS scope on your servers.
  </Card>
  <Card title="Payment Links" icon="fa-duotone fa-link">
    Generate shareable payment links with configurable expiry — no integration required on the customer side.
  </Card>
  <Card title="Webhook Notifications" icon="fa-duotone fa-bell">
    Receive real-time payment status notifications via signed webhooks with retry logic.
  </Card>
</Cards>

<br />

## Environment URLs

| Environment   | Base URL                         |
| ------------- | -------------------------------- |
| UAT (Sandbox) | `https://<uat-host>:8085/s2s/v1` |
| Production    | `https://<prod-host>/s2s/v1`     |

> fa-solid fa-triangle-exclamation
>
> Never use production credentials in the UAT environment, or UAT credentials against production endpoints. Maintain separate key pairs for each environment.

<br />

## Supported Payment Methods

| Method      | `paymentMode` | `paymentCode`           | Description                                 |
| ----------- | ------------- | ----------------------- | ------------------------------------------- |
| UPI Collect | `UPI`         | `UPII`                  | Push collect request to customer's UPI app  |
| UPI Intent  | `UPI`         | `UPIC`                  | Deep-link to UPI app on the same device     |
| Dynamic QR  | `UPI`         | `DQR`                   | Generate scannable QR for UPI payment       |
| Net Banking | `NB`          | Bank code (e.g. `HDFC`) | Redirect to bank's net banking portal       |
| Credit Card | `CC`          | `CC`                    | Accept Visa, Mastercard, RuPay credit cards |
| Debit Card  | `DC`          | `DC`                    | Accept Visa, Mastercard, RuPay debit cards  |

<br />

## API Reference

<Cards>
  <Card title="Verify VPA" href="/reference/verify-vpa" icon="fa-duotone fa-badge-check">
    Validate a UPI VPA before initiating a UPI Collect payment.
  </Card>
  <Card title="Netbanking List" href="/reference/netbanking-list" icon="fa-duotone fa-building-columns">
    Retrieve the list of available net banking options.
  </Card>
  <Card title="Enabled Instruments" href="/reference/enabled-instruments" icon="fa-duotone fa-credit-card">
    List all payment instruments enabled for your merchant account.
  </Card>
  <Card title="Initiate Transaction" href="/reference/initiate-transaction" icon="fa-duotone fa-money-check-dollar">
    Create a new payment transaction across any supported payment mode.
  </Card>
  <Card title="Transaction Status" href="/reference/transaction-status" icon="fa-duotone fa-list-check">
    Query the current status of any transaction by RRN or transaction ID.
  </Card>
  <Card title="Checkout Intent" href="/reference/checkout-intent" icon="fa-duotone fa-cart-shopping">
    Create a hosted checkout session and redirect the customer.
  </Card>
  <Card title="Checkout Payment Link" href="/reference/checkout-payment-link" icon="fa-duotone fa-link">
    Generate a shareable payment link with configurable expiry.
  </Card>
</Cards>

<br />

## Quick Navigation

<Cards columns={2}>
  <Card title="Quick Start Guide" href="/docs/quickstart" icon="fa-duotone fa-rocket">
    Get your first test transaction running in under 10 minutes.
  </Card>
  <Card title="Authentication" href="/docs/authentication" icon="fa-duotone fa-key">
    Understand HMAC-SHA256 checksum generation with code samples in 5 languages.
  </Card>
  <Card title="Encryption Guide" href="/docs/encryption" icon="fa-duotone fa-lock">
    Encrypt requests and decrypt responses using AES-256.
  </Card>
  <Card title="API Flows" href="/docs/api-flows" icon="fa-duotone fa-diagram-project">
    End-to-end flow diagrams for every payment method.
  </Card>
  <Card title="Error Handling" href="/docs/error-handling" icon="fa-duotone fa-circle-exclamation">
    HTTP status codes, error payloads, and resolution steps.
  </Card>
  <Card title="Webhooks" href="/docs/webhooks" icon="fa-duotone fa-webhook">
    Configure and verify real-time payment notifications.
  </Card>
  <Card title="Best Practices" href="/docs/security-best-practices" icon="fa-duotone fa-shield-check">
    Security, retry strategy, idempotency, and reconciliation.
  </Card>
  <Card title="Go-Live Checklist" href="/docs/production-checklist" icon="fa-duotone fa-clipboard-check">
    Everything you need to verify before going live in production.
  </Card>
</Cards>

<br />

## Checksum Formulas Reference

| API                  | Checksum Formula                                               |
| -------------------- | -------------------------------------------------------------- |
| Verify VPA           | `mid\|VPA_VERIFY\|vpa\|salt`                                   |
| Netbanking List      | `mid\|NETBANKING_LIST\|salt`                                   |
| Enabled Instruments  | `mid\|ENABLED_INSTRUMENTS\|salt`                               |
| Initiate Transaction | `mid\|orderNo\|rrn\|txnAmount\|paymentMode\|paymentCode\|salt` |
| Transaction Status   | `mid\|rrn\|txnId\|salt`                                        |

> fa-solid fa-circle-info
>
> The `salt` value is a secret shared between your server and the gateway. Never expose it in client-side code, mobile apps, or browser JavaScript.

Use the S2S Payment Gateway APIs to verify payment details, initiate transactions, check transaction status, and create checkout payment flows.

## Base URLs

| Environment | Base URL                         |
| ----------- | -------------------------------- |
| UAT         | `https://<uat-host>:8085/s2s/v1` |
| Production  | `https://<prod-host>/s2s/v1`     |

## Authentication and encryption

Authenticate requests with an HMAC-SHA256 checksum. Send merchant identification with the `X-Merchant-Id` header where required, and include `X-Checksum` for endpoints that require a checksum header.

Requests and responses use AES-256 encryption.

<br />

## API Reference

<Cards>
  <Card title="Verify VPA" href="/reference/verify-vpa" icon="fa-duotone fa-badge-check">Check whether a UPI VPA is valid.</Card>

<Card title="Netbanking List" href="/reference/netbanking-list" icon="fa-duotone fa-building-columns">Retrieve available netbanking options.</Card>

<Card title="Enabled Instruments" href="/reference/enabled-instruments" icon="fa-duotone fa-credit-card">List enabled payment instruments.</Card>

<Card title="Initiate Transaction" href="/reference/initiate-transaction" icon="fa-duotone fa-money-check-dollar">Start a payment transaction.</Card>

<Card title="Transaction Status" href="/reference/transaction-status" icon="fa-duotone fa-list-check">Check the current status of a transaction.</Card>

<Card title="Checkout Intent" href="/reference/checkout-intent" icon="fa-duotone fa-cart-shopping">Create a checkout payment intent.</Card>

<Card title="Checkout Payment Link" href="/reference/checkout-payment-link" icon="fa-duotone fa-link">Create a checkout payment link.</Card> </Cards>

<br />

## Checksum formulas

| API                  | Formula |                      |        |           |             |             |        |
| -------------------- | ------- | -------------------- | ------ | --------- | ----------- | ----------- | ------ |
| Verify VPA           | \`mid   | VPA\_VERIFY          | vpa    | salt\`    |             |             |        |
| Netbanking List      | \`mid   | NETBANKING\_LIST     | salt\` |           |             |             |        |
| Enabled Instruments  | \`mid   | ENABLED\_INSTRUMENTS | salt\` |           |             |             |        |
| Initiate Transaction | \`mid   | orderNo              | rrn    | txnAmount | paymentMode | paymentCode | salt\` |
| Transaction Status   | \`mid   | rrn                  | txnId  | salt\`    |             |             |        |

## Supported payment modes

- `UPI` — UPI payments
- `NB` — Net Banking
- `CC` — Credit Card
- `DC` — Debit Card

<br />
