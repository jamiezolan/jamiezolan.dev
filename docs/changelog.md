# Changelog

Notable changes to the Pudgy API and its documentation. The API is versioned in the URL (`/v1`); breaking changes ship under a new version, never silently.

## 2026-05-30

- **Added** the [Data classification](security/data-classification.md) standard with field-level masking guidance.
- **Added** cursor-based pagination to all list endpoints; offset paging is deprecated.
- **Changed** the error envelope to include a `doc_url` linking each error to its explanation.

## 2026-04-12

- **Added** `subscription.renewed` and `invoice.payment_failed` webhook events.
- **Improved** idempotency conflict responses to distinguish "in progress" from "key reused with a different body".

## 2026-03-01

- **Added** the sandbox environment and deterministic [test cards](get-started/sandbox.md#test-cards).
- **Added** webhook signature verification (`Pudgy-Signature`).
