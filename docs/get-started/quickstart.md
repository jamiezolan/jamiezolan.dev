# Quickstart

This guide takes you from zero to a successful test charge. It should take about five minutes.

## Before you begin

You need a sandbox API key. Sandbox keys start with `sk_test_` and only ever touch simulated money. Keep them out of source control and out of client-side code.

```bash
export PUDGY_KEY="sk_test_your_key_here"
```

## 1. Verify your key

A quick read-only call confirms the key works and the network path is open.

```bash
curl https://api.pudgypay.dev/v1/account \
  -H "Authorization: Bearer $PUDGY_KEY"
```

A `200 OK` with your account object means you're ready. A `401` means the key is missing or wrong — see [Authentication](../concepts/authentication.md).

## 2. Create a charge

This charges a test card 20.00 USD. Amounts are always in the currency's smallest unit, so 20.00 USD is `2000`.

```bash
curl https://api.pudgypay.dev/v1/charges \
  -H "Authorization: Bearer $PUDGY_KEY" \
  -H "Idempotency-Key: $(uuidgen)" \
  -d amount=2000 \
  -d currency=usd \
  -d source=tok_visa \
  -d "description=Quickstart test charge"
```

The `Idempotency-Key` header is optional here but you should treat it as mandatory in real code. It makes the request safe to retry without double-charging — see [Idempotency](../concepts/idempotency.md).

A successful response:

```json
{
  "id": "ch_3Nf8k2eZvKYlo2C1",
  "object": "charge",
  "amount": 2000,
  "currency": "usd",
  "status": "succeeded",
  "paid": true,
  "description": "Quickstart test charge",
  "created": "2026-05-30T17:42:11Z"
}
```

## 3. Retrieve the charge

Use the `id` from the previous response to read the charge back.

```bash
curl https://api.pudgypay.dev/v1/charges/ch_3Nf8k2eZvKYlo2C1 \
  -H "Authorization: Bearer $PUDGY_KEY"
```

## What just happened

You authenticated with a bearer token, created a resource with a typed JSON response, and read it back by ID. Every other resource in the API — customers, refunds, payouts — follows the same pattern.

## Next steps

<div class="grid cards" markdown>

-   **Make it production-grade**

    Add idempotency keys and structured error handling.

    [:octicons-arrow-right-24: Idempotency](../concepts/idempotency.md)

-   **React to events**

    Stop polling. Let Pudgy notify you when a charge settles or fails.

    [:octicons-arrow-right-24: Webhooks](../concepts/webhooks.md)

-   **Build a real flow**

    Walk through accepting a one-time payment from a customer.

    [:octicons-arrow-right-24: Accept a payment](../guides/accept-a-payment.md)

</div>
