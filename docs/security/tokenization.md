# Tokenization

Tokenization replaces a sensitive card number with a non-sensitive stand-in — a **token** — that is useless to an attacker but lets you charge the card again. It's the mechanism behind almost every claim on the other security pages: smaller PCI scope, no PAN in responses, safe stored cards.

!!! tip "Pudgy says"
    The real card goes into my burrow, where it's safe and out of sight. You get a token — a little chewed-up IOU — that means nothing to anyone but me. Lose it and a thief gets… a soggy IOU.

## The exchange

1. The browser sends raw card details directly to Pudgy using your **publishable** key.
2. Pudgy stores the card in its secure vault and returns a **single-use token** (`tok_...`).
3. Your server uses that token to create a charge or attach the card to a customer.

The raw card number exists in your stack for exactly zero milliseconds. It went from the browser to Pudgy without passing through your server.

```javascript
// Browser — publishable key, safe to ship
const { token } = await pudgy.createToken(cardElement);
// You forward token.id to your server; the PAN stays behind.
```

## Single-use tokens vs. saved cards

- A **single-use token** (`tok_...`) is consumed by one charge and then expires. Use it for one-time payments.
- To charge a customer again later — subscriptions, one-click checkout — attach the card to a **customer**, which produces a durable `card_...` reference stored in Pudgy's vault. You store only the `customer` and `card` IDs.

```bash
# Attach a card to a customer for reuse
curl https://api.pudgypay.dev/v1/customers/cus_9aZ.../cards \
  -H "Authorization: Bearer $PUDGY_KEY" \
  -d source=tok_visa
```

## Why this matters

| Without tokenization | With tokenization |
|---|---|
| Your database holds card numbers | Your database holds opaque IDs |
| A breach exposes live cards | A breach exposes useless tokens |
| Your servers are in full PCI scope | Scope shrinks dramatically |
| You must encrypt, rotate, and audit PANs | Pudgy's vault does it |

The token is meaningless outside Pudgy: it can't be reversed into a card number and it can't be used against another merchant's account.
