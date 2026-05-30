# Errors

Pudgy returns standard HTTP status codes, and every error body has the same shape. You should branch on the `code` field, never on the human-readable `message`.

## The error envelope

```json
{
  "error": {
    "type": "card_error",
    "code": "card_declined",
    "message": "The card was declined.",
    "param": "source",
    "doc_url": "https://docs.pudgypay.dev/errors/card_declined"
  }
}
```

| Field | Use it for |
|---|---|
| `type` | The broad category (see below). Good for metrics and routing. |
| `code` | The specific, stable reason. **Branch on this.** |
| `message` | Display or logging. Wording may change; never parse it. |
| `param` | Which request field caused a validation error, if any. |
| `doc_url` | A link your engineers can follow straight to the explanation. |

## Status codes

| Status | Meaning | What you should do |
|---|---|---|
| `200` | Success | Proceed |
| `400` | Malformed request | Fix the request; do not retry unchanged |
| `401` | Bad or missing key | Check credentials — [Authentication](authentication.md) |
| `402` | Payment failed (e.g. declined) | Show the customer a recoverable message |
| `404` | No such object | Check the ID |
| `409` | Conflict (idempotency) | See [Idempotency](idempotency.md) |
| `429` | Rate limited | Back off and retry per the `Retry-After` header |
| `5xx` | Something broke on our side | Retry with backoff and the same idempotency key |

## Error types

- **`authentication_error`** — the key is missing, malformed, or revoked.
- **`invalid_request_error`** — a parameter is wrong; check `param`.
- **`card_error`** — the card was declined or is unusable. The most common type in production and the one most often mishandled.
- **`rate_limit_error`** — you're sending too fast.
- **`api_error`** — a rare internal failure. Safe to retry.

!!! tip "Pudgy says"
    A decline isn't a crash. I just sniffed the card and turned up my nose. Tell the human politely and let them try another — don't keep shoving the same card at me.

## Handle declines, don't crash on them

A decline is a normal business outcome, not an exception. Surface it to the customer and let them recover:

```python
resp = create_charge(...)
if resp.status_code == 402:
    code = resp.json()["error"]["code"]
    if code == "insufficient_funds":
        prompt_customer("That card was declined. Try another card.")
    elif code == "expired_card":
        prompt_customer("That card has expired.")
    else:
        prompt_customer("We couldn't process that card.")
    # No retry: the same card will be declined again.
```

The rule: retry `5xx` and `429` with backoff; never auto-retry a `402` decline.
