# Accept a one-time payment

This guide builds a complete, correct one-time payment: tokenize the card in the browser, charge it on your server, and confirm the result through a webhook. It assumes you've finished the [Quickstart](../get-started/quickstart.md).

## The flow at a glance

1. The browser collects card details and exchanges them for a single-use **token** using your publishable key. Raw card data never reaches your server.
2. Your server creates a **charge** with that token, your secret key, and an idempotency key.
3. Pudgy sends a **`charge.succeeded`** webhook, which your server treats as the source of truth.

## 1. Tokenize in the browser

Card data is captured by Pudgy's hosted field and returned as a token. Because you use the publishable key here, this code is safe to ship to the client.

```javascript
const token = await pudgy.createToken(cardElement); // pk_test_...
// Send only token.id to your server — never the card number.
await fetch("/api/checkout", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ token: token.id, amount: 2000 }),
});
```

Keeping raw card numbers off your servers is what keeps you out of the most demanding parts of PCI scope. See [PCI scope](../security/pci-scope.md).

## 2. Charge on the server

```python
import uuid, requests

def checkout(token_id, amount):
    return requests.post(
        "https://api.pudgypay.dev/v1/charges",
        headers={
            "Authorization": f"Bearer {SECRET_KEY}",
            "Idempotency-Key": str(uuid.uuid4()),
        },
        data={"amount": amount, "currency": "usd", "source": token_id},
        timeout=10,
    )
```

Handle the response by category, not by guesswork:

```python
resp = checkout(token_id, 2000)
if resp.ok:
    return {"status": "processing"}        # don't fulfill yet — wait for the webhook
elif resp.status_code == 402:
    return {"status": "declined",
            "code": resp.json()["error"]["code"]}
else:
    resp.raise_for_status()
```

## 3. Fulfill on the webhook, not the response

It's tempting to ship the goods the instant the charge call returns `200`. Don't. The authoritative signal that money settled is the `charge.succeeded` event. Fulfilling on the synchronous response alone leaves you exposed to edge cases where the response is lost but the charge succeeded.

```python
@app.post("/webhooks/pudgy")
def handle(request):
    event = verify_and_parse(request)      # see Handle webhooks
    if event["type"] == "charge.succeeded":
        fulfill_order(event["data"]["object"]["id"])
    return Response(status=200)
```

See [Handle webhooks](handle-webhooks.md) for the verification and idempotency details.

## Checklist

- [x] Card tokenized client-side with a publishable key
- [x] Charge created server-side with a secret key
- [x] Idempotency key on the charge request
- [x] Declines handled as a normal outcome
- [x] Fulfillment driven by the `charge.succeeded` webhook
