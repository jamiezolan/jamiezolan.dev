# Handle webhooks

This guide is the practical companion to the [Webhooks concept](../concepts/webhooks.md). It walks through standing up an endpoint that is secure and correct under retries.

## 1. Expose an endpoint

Register a public HTTPS URL in the dashboard (or via the API). Pudgy will `POST` events to it. In local development, tunnel to your machine:

```bash
pudgy listen --forward-to localhost:4242/webhooks
```

This also prints your endpoint's signing secret (`whsec_...`), which you'll need to verify signatures.

## 2. Verify, enqueue, acknowledge

The handler should do three things and nothing more on the request path:

```python
@app.post("/webhooks")
def webhook(request):
    payload = request.get_data()                     # raw bytes
    sig = request.headers["Pudgy-Signature"]

    try:
        verify(payload, sig, WEBHOOK_SECRET)         # 1. authenticate
    except ValueError:
        return Response(status=400)                  # reject forgeries

    event = json.loads(payload)
    if not already_processed(event["id"]):           # 2. dedupe
        enqueue(event)                               # 3. work happens elsewhere
        mark_processed(event["id"])

    return Response(status=200)                      # ack fast
```

The `verify` function is in the [Webhooks concept](../concepts/webhooks.md#always-verify-the-signature).

## 3. Test it before you trust it

Fire a real event into your endpoint from the sandbox:

```bash
curl https://api.pudgypay.dev/v1/test/events \
  -H "Authorization: Bearer $PUDGY_KEY" \
  -d type=charge.succeeded
```

Confirm that your endpoint accepts the valid event, rejects a tampered one with `400`, and processes a duplicate exactly once.

## Failure handling reference

- **Non-`2xx` response** → Pudgy retries with exponential backoff for up to 72 hours.
- **Endpoint down for a while** → events queue and replay when you recover; design for out-of-order, duplicated delivery.
- **Persistent failures** → the endpoint is disabled and you're notified. Re-enable it after a fix and replay missed events from the dashboard.
