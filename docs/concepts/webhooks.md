# Webhooks

Webhooks let Pudgy push events to your server the moment something changes, so you stop polling. When a charge succeeds, a dispute opens, or a subscription renews, Pudgy sends an HTTP `POST` to a URL you control.

## The event object

```json
{
  "id": "evt_1P2x...",
  "object": "event",
  "type": "charge.succeeded",
  "created": "2026-05-30T17:42:11Z",
  "data": {
    "object": { "id": "ch_3Nf...", "object": "charge", "status": "succeeded" }
  }
}
```

`data.object` contains the full resource in its state at the time of the event.

## Common event types

| Event | Fires when |
|---|---|
| `charge.succeeded` | A charge is paid |
| `charge.failed` | A charge is declined |
| `charge.refunded` | A refund is issued |
| `customer.created` | A customer is created |
| `subscription.renewed` | A recurring subscription bills successfully |
| `dispute.created` | A cardholder disputes a charge |

!!! tip "Pudgy says"
    When something happens, I squeak at you — no need to keep poking the cage to check. But verify the squeak is really mine. A cat can do a passable hamster impression, and the signature is how you tell us apart.

## Always verify the signature

Anyone can POST JSON to a public URL. The signature proves the request actually came from Pudgy and was not tampered with in transit.

Each delivery includes a `Pudgy-Signature` header containing a timestamp and an HMAC-SHA256 of the timestamp plus the raw request body, keyed with your endpoint's signing secret (`whsec_...`).

```python
import hmac, hashlib, time

def verify(payload: bytes, header: str, secret: str, tolerance=300):
    parts = dict(p.split("=", 1) for p in header.split(","))
    ts, sig = parts["t"], parts["v1"]

    # 1. Reject anything too old — blocks replayed deliveries.
    if abs(time.time() - int(ts)) > tolerance:
        raise ValueError("timestamp outside tolerance")

    # 2. Recompute the HMAC over `timestamp.rawbody` and compare.
    signed = f"{ts}.".encode() + payload
    expected = hmac.new(secret.encode(), signed, hashlib.sha256).hexdigest()
    if not hmac.compare_digest(expected, sig):   # constant-time compare
        raise ValueError("signature mismatch")
```

!!! warning "Verify against the raw body"
    Compute the signature over the exact bytes you received, before any JSON parsing or framework middleware reformats them. Re-serializing the body changes the bytes and the signature will never match.

## Respond fast, work later

Return `2xx` as soon as you've stored the event. Do the real work asynchronously.

- Pudgy considers any non-`2xx` response a failure and **retries with exponential backoff** for up to 72 hours.
- Slow handlers cause timeouts, which look like failures, which trigger retries, which can stampede your system.

The reliable pattern: verify the signature, enqueue the event, return `200`. A worker processes the queue.

## Make handlers idempotent

Because of retries, you **will** receive the same event more than once. Deduplicate on the event `id`:

```python
if already_processed(event["id"]):
    return Response(status=200)   # ack and ignore the duplicate
mark_processed(event["id"])
handle(event)
```

Storing processed event IDs is the simplest defense and it makes your system correct under retries.
