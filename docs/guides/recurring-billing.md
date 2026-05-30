# Set up recurring billing

Recurring billing bills a customer automatically on a schedule. The model has three objects: a **product** (what you sell), a **price** (how much and how often), and a **subscription** (a customer attached to a price).

## 1. Create a product and price

```bash
curl https://api.pudgypay.dev/v1/products \
  -H "Authorization: Bearer $PUDGY_KEY" \
  -d name="Pro plan"

curl https://api.pudgypay.dev/v1/prices \
  -H "Authorization: Bearer $PUDGY_KEY" \
  -d product=prod_Pro \
  -d unit_amount=1500 \
  -d currency=usd \
  -d "recurring[interval]=month"
```

## 2. Create a customer with a saved card

```bash
curl https://api.pudgypay.dev/v1/customers \
  -H "Authorization: Bearer $PUDGY_KEY" \
  -d email="dev@example.com" \
  -d source=tok_visa
```

## 3. Subscribe the customer

```bash
curl https://api.pudgypay.dev/v1/subscriptions \
  -H "Authorization: Bearer $PUDGY_KEY" \
  -d customer=cus_9aZ... \
  -d price=price_Pro_Monthly
```

Pudgy now bills the customer every month and emits an event on each cycle.

## Handle the billing lifecycle

Subscriptions live or die by how you handle the events. Subscribe to these and act on them:

| Event | What it means | Typical action |
|---|---|---|
| `subscription.renewed` | A cycle billed successfully | Extend access |
| `invoice.payment_failed` | A renewal was declined | Start dunning; warn the customer |
| `subscription.canceled` | The subscription ended | Revoke access at period end |

!!! warning "Failed renewals are the hard part"
    Cards expire and funds run short. A subscription business that ignores `invoice.payment_failed` quietly bleeds revenue. Implement a dunning sequence: retry on a schedule, email the customer, and downgrade access only after retries are exhausted.
