---
title: Production Go-Live Checklist
excerpt: >-
  Production readiness checklist for credentials, authentication, encryption,
  webhooks, reliability, compliance, and reconciliation.
---
Use this checklist to verify your integration before switching from UAT to production credentials.

## Credentials and Access

- [ ] Production `mid` is configured only on production servers.
- [ ] Production `salt`, encryption key, and IV are stored in a secret manager.
- [ ] UAT credentials are not present in production environment variables.
- [ ] Secrets are not committed to source control, container images, CI logs, or mobile apps.

<br />

## Authentication

- [ ] Each endpoint uses the correct checksum formula.
- [ ] Checksum generation uses the exact field values sent in the request.
- [ ] Amount formatting is consistent between checksum and payload.
- [ ] Checksum failures are logged with correlation IDs, not raw salt values.

<br />

## Encryption

- [ ] All request bodies are AES-256 encrypted.
- [ ] All encrypted responses are decrypted and validated before processing.
- [ ] Plaintext payloads are not logged in production.
- [ ] PAN, CVV, VPA, phone, and email fields are masked in logs.

<br />

## Payment Flows

- [ ] Verify VPA tested for valid and invalid UPI IDs.
- [ ] UPI Collect tested for success, failure, pending, and timeout states.
- [ ] Dynamic QR tested on multiple UPI apps.
- [ ] Net Banking redirect and return URL tested for all enabled banks.
- [ ] Card payments tested for success, issuer decline, 3DS challenge, and timeout.
- [ ] Hosted Checkout redirect and callback flow tested.
- [ ] Payment Link expiry and reuse behavior tested.

<br />

## Webhooks

- [ ] Webhook endpoint is public HTTPS and returns HTTP `200` within 5 seconds.
- [ ] Webhook signature validation is enabled.
- [ ] Duplicate webhook handling is idempotent.
- [ ] Webhook payloads are stored for audit.
- [ ] Transaction Status API is used when webhook delivery fails.

<br />

## Reliability

- [ ] Network timeouts trigger Transaction Status lookup before retrying payment.
- [ ] `PENDING` transactions remain pending until final status is received.
- [ ] Reconciliation job runs at least daily.
- [ ] Failed reconciliation records are escalated to operations.

<br />

## Security and Compliance

- [ ] TLS 1.2 or higher is enforced.
- [ ] Card data handling is PCI-DSS reviewed if card details touch merchant servers.
- [ ] CVV is never stored.
- [ ] Access to production secrets is limited and audited.
- [ ] API logs include request IDs, order numbers, and transaction IDs, but not secrets.

<br />

## Go-Live Signoff

| Area | Owner | Status |
| --- | --- | --- |
| Business onboarding complete | Merchant Ops | Pending |
| Production credentials issued | Gateway Ops | Pending |
| Integration testing passed | Engineering | Pending |
| Webhook testing passed | Engineering | Pending |
| Reconciliation tested | Finance / Ops | Pending |
| Security review complete | Security | Pending |
| Production switch approved | Business Owner | Pending |
Use this checklist to verify your integration before switching from UAT to production credentials.

## Credentials and Access

- [ ] Production `mid` is configured only on production servers.
- [ ] Production `salt`, encryption key, and IV are stored in a secret manager.
- [ ] UAT credentials are not present in production environment variables.
- [ ] Secrets are not committed to source control, container images, CI logs, or mobile apps.

<br />

## Authentication

- [ ] Each endpoint uses the correct checksum formula.
- [ ] Checksum generation uses the exact field values sent in the request.
- [ ] Amount formatting is consistent between checksum and payload.
- [ ] Checksum failures are logged with correlation IDs, not raw salt values.

<br />

## Encryption

- [ ] All request bodies are AES-256 encrypted.
- [ ] All encrypted responses are decrypted and validated before processing.
- [ ] Plaintext payloads are not logged in production.
- [ ] PAN, CVV, VPA, phone, and email fields are masked in logs.

<br />

## Payment Flows

- [ ] Verify VPA tested for valid and invalid UPI IDs.
- [ ] UPI Collect tested for success, failure, pending, and timeout states.
- [ ] Dynamic QR tested on multiple UPI apps.
- [ ] Net Banking redirect and return URL tested for all enabled banks.
- [ ] Card payments tested for success, issuer decline, 3DS challenge, and timeout.
- [ ] Hosted Checkout redirect and callback flow tested.
- [ ] Payment Link expiry and reuse behavior tested.

<br />

## Webhooks

- [ ] Webhook endpoint is public HTTPS and returns HTTP `200` within 5 seconds.
- [ ] Webhook signature validation is enabled.
- [ ] Duplicate webhook handling is idempotent.
- [ ] Webhook payloads are stored for audit.
- [ ] Transaction Status API is used when webhook delivery fails.

<br />

## Reliability

- [ ] Network timeouts trigger Transaction Status lookup before retrying payment.
- [ ] `PENDING` transactions remain pending until final status is received.
- [ ] Reconciliation job runs at least daily.
- [ ] Failed reconciliation records are escalated to operations.

<br />

## Security and Compliance

- [ ] TLS 1.2 or higher is enforced.
- [ ] Card data handling is PCI-DSS reviewed if card details touch merchant servers.
- [ ] CVV is never stored.
- [ ] Access to production secrets is limited and audited.
- [ ] API logs include request IDs, order numbers, and transaction IDs, but not secrets.

<br />

## Go-Live Signoff

| Area | Owner | Status |
| --- | --- | --- |
| Business onboarding complete | Merchant Ops | Pending |
| Production credentials issued | Gateway Ops | Pending |
| Integration testing passed | Engineering | Pending |
| Webhook testing passed | Engineering | Pending |
| Reconciliation tested | Finance / Ops | Pending |
| Security review complete | Security | Pending |
| Production switch approved | Business Owner | Pending |
Use this checklist to verify your integration before switching from UAT to production credentials.

## Credentials and Access

- [ ] Production `mid` is configured only on production servers.
- [ ] Production `salt`, encryption key, and IV are stored in a secret manager.
- [ ] UAT credentials are not present in production environment variables.
- [ ] Secrets are not committed to source control, container images, CI logs, or mobile apps.

<br />

## Authentication

- [ ] Each endpoint uses the correct checksum formula.
- [ ] Checksum generation uses the exact field values sent in the request.
- [ ] Amount formatting is consistent between checksum and payload.
- [ ] Checksum failures are logged with correlation IDs, not raw salt values.

<br />

## Encryption

- [ ] All request bodies are AES-256 encrypted.
- [ ] All encrypted responses are decrypted and validated before processing.
- [ ] Plaintext payloads are not logged in production.
- [ ] PAN, CVV, VPA, phone, and email fields are masked in logs.

<br />

## Payment Flows

- [ ] Verify VPA tested for valid and invalid UPI IDs.
- [ ] UPI Collect tested for success, failure, pending, and timeout states.
- [ ] Dynamic QR tested on multiple UPI apps.
- [ ] Net Banking redirect and return URL tested for all enabled banks.
- [ ] Card payments tested for success, issuer decline, 3DS challenge, and timeout.
- [ ] Hosted Checkout redirect and callback flow tested.
- [ ] Payment Link expiry and reuse behavior tested.

<br />

## Webhooks

- [ ] Webhook endpoint is public HTTPS and returns HTTP `200` within 5 seconds.
- [ ] Webhook signature validation is enabled.
- [ ] Duplicate webhook handling is idempotent.
- [ ] Webhook payloads are stored for audit.
- [ ] Transaction Status API is used when webhook delivery fails.

<br />

## Reliability

- [ ] Network timeouts trigger Transaction Status lookup before retrying payment.
- [ ] `PENDING` transactions remain pending until final status is received.
- [ ] Reconciliation job runs at least daily.
- [ ] Failed reconciliation records are escalated to operations.

<br />

## Security and Compliance

- [ ] TLS 1.2 or higher is enforced.
- [ ] Card data handling is PCI-DSS reviewed if card details touch merchant servers.
- [ ] CVV is never stored.
- [ ] Access to production secrets is limited and audited.
- [ ] API logs include request IDs, order numbers, and transaction IDs, but not secrets.

<br />

## Go-Live Signoff

| Area | Owner | Status |
| --- | --- | --- |
| Business onboarding complete | Merchant Ops | Pending |
| Production credentials issued | Gateway Ops | Pending |
| Integration testing passed | Engineering | Pending |
| Webhook testing passed | Engineering | Pending |
| Reconciliation tested | Finance / Ops | Pending |
| Security review complete | Security | Pending |
| Production switch approved | Business Owner | Pending |