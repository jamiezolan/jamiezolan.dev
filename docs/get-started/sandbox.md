# The sandbox

Think of it as Pudgy's playpen: a complete, isolated copy of the API where no real seeds are ever spent. Use it for development, automated tests, and demos.

## Test keys vs. live keys

| | Test mode | Live mode |
|---|---|---|
| Key prefix | `sk_test_` | `sk_live_` |
| Money movement | Simulated only | Real |
| Webhooks | Delivered to your test endpoints | Delivered to your live endpoints |
| Data | Cleared periodically | Retained per your data policy |

The two environments share no data. An object created with a test key is invisible to a live key and vice versa.

## Test cards

The sandbox recognizes special card tokens that force a specific outcome, so you can exercise both happy and unhappy paths deterministically.

| Token | Behavior |
|---|---|
| `tok_visa` | Succeeds |
| `tok_mastercard` | Succeeds |
| `tok_chargeDeclined` | Declined with `card_declined` |
| `tok_insufficientFunds` | Declined with `insufficient_funds` |
| `tok_expiredCard` | Declined with `expired_card` |
| `tok_processingError` | Returns a `502` you can retry |

Testing failures is not optional. The most common production incident in payments is mishandling a decline you never tested.

## Triggering webhook events

In test mode you can fire any event on demand rather than waiting for a real state change:

```bash
curl https://api.pudgypay.dev/v1/test/events \
  -H "Authorization: Bearer $PUDGY_KEY" \
  -d type=charge.succeeded
```

This delivers a fully-formed event to your registered test endpoints so you can validate signature checking and handler logic. See [Handle webhooks](../guides/handle-webhooks.md).
