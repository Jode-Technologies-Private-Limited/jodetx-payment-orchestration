---
title: S2S Payment Gateway
excerpt: >-
  Use the S2S Payment Gateway APIs to verify VPAs, list payment instruments,
  initiate transactions, check transaction status, and create checkout payment
  flows.
hidden: false
---
Use the S2S Payment Gateway APIs to verify payment details, initiate transactions, check transaction status, and create checkout payment flows.

## Base URLs

| Environment | Base URL |
| --- | --- |
| UAT | `https://<uat-host>:8085/s2s/v1` |
| Production | `https://<prod-host>/s2s/v1` |

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

  <Card title="Checkout Payment Link" href="/reference/checkout-payment-link" icon="fa-duotone fa-link">Create a checkout payment link.</Card>
</Cards>

<br />

## Checksum formulas

| API | Formula |
| --- | --- |
| Verify VPA | `mid|VPA_VERIFY|vpa|salt` |
| Netbanking List | `mid|NETBANKING_LIST|salt` |
| Enabled Instruments | `mid|ENABLED_INSTRUMENTS|salt` |
| Initiate Transaction | `mid|orderNo|rrn|txnAmount|paymentMode|paymentCode|salt` |
| Transaction Status | `mid|rrn|txnId|salt` |

## Supported payment modes

- `UPI` — UPI payments
- `NB` — Net Banking
- `CC` — Credit Card
- `DC` — Debit Card