# Idempotency

Networks fail in the worst possible way: after your request succeeded but before you heard back. If you retry blindly, you charge the customer twice. Idempotency keys make a retry safe.

!!! tip "Pudgy says"
    I stuff each seed into my cheek pouch exactly once. Hand me the same seed twice and I just show you the one already in there — I never double-stuff. The idempotency key is the label on the seed.

## How it works

Send a unique `Idempotency-Key` header with any state-changing request:

```bash
curl https://api.pudgypay.dev/v1/charges \
  -H "Authorization: Bearer $PUDGY_KEY" \
  -H "Idempotency-Key: a1b2c3d4-e5f6-4789-a0b1-c2d3e4f5a6b7" \
  -d amount=2000 -d currency=usd -d source=tok_visa
```

The first time Pudgy sees a key, it executes the request and stores the result against that key. If the same key arrives again — your retry — Pudgy returns the **stored original response** instead of running the operation a second time. One key, one effect, no matter how many times you send it.

## Choosing a key

- Generate a **UUID v4** (or any value with enough entropy to be globally unique).
- Generate it **once per logical operation**, before the first attempt, and reuse it for every retry of that same operation.
- Do **not** reuse a key across different operations — a second, genuinely different charge needs its own key.

A useful mental model: the key identifies the *intent* ("charge this cart now"), not the HTTP request.

## Lifetime and edge cases

!!! info "Keys expire after 24 hours"
    Pudgy retains idempotency keys for 24 hours. Retries within that window are safe. After it, the key is forgotten and a request using it executes as new — so don't rely on it for long-term deduplication.

A few behaviors worth knowing:

- **Same key, identical request → original response replayed.** This is the happy path for a retry.
- **Same key, *different* request body → `409 Conflict`.** Pudgy refuses to let one key stand for two different operations. This usually means a bug in how you generate keys.
- **Request still in flight → `409 Conflict`** with code `idempotency_in_progress`. Wait and retry; don't fire concurrent requests on one key.

## A safe retry loop

```python
import uuid, time, requests

def charge(amount, source, key=None):
    key = key or str(uuid.uuid4())   # generated once, reused on retry
    for attempt in range(3):
        resp = requests.post(
            "https://api.pudgypay.dev/v1/charges",
            headers={"Authorization": f"Bearer {API_KEY}",
                     "Idempotency-Key": key},   # (1)!
            data={"amount": amount, "currency": "usd", "source": source},
            timeout=10,
        )
        if resp.status_code < 500:
            return resp.json()       # success or a real client error
        time.sleep(2 ** attempt)     # back off, then retry the SAME key
    resp.raise_for_status()
```

1.  The same key is reused across every attempt, so at most one charge is ever created.
