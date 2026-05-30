# Data classification

Not all payment data is equally sensitive, and treating it as if it were leads to two failures: over-protecting low-risk fields wastes effort, and under-protecting high-risk fields invites breach and regulatory exposure. Pudgy classifies every field it returns into one of four tiers so you can apply controls proportionally.

## The tiers

| Tier | Definition | Example fields | Default handling |
|---|---|---|---|
| **Restricted** | Data whose exposure is a reportable security event | Full PAN, CVV, magnetic-stripe data | Never stored by you; never returned by the API |
| **Confidential** | Identifies a person or enables fraud if combined | Email, billing address, last 4 digits, cardholder name | Encrypt at rest; mask in logs and UIs |
| **Internal** | Operational data with limited sensitivity | Charge IDs, timestamps, statuses, amounts | Access-controlled; safe in internal systems |
| **Public** | Data with no confidentiality requirement | API version, currency codes, documentation | No restriction |

## What the API will and won't return

Pudgy is designed so you rarely hold Restricted data at all:

- A raw PAN is **accepted once** during tokenization and then **never returned**. After tokenization you only ever see a `token` and a masked representation.
- **CVV is never stored** by anyone, ever — not by you and not by Pudgy. It may be used to authorize a single transaction and is then discarded.
- Card objects expose `last4`, `brand`, and `exp_month`/`exp_year` (Confidential), which are enough to display a card to its owner without holding the number.

```json
{
  "id": "card_1Lx...",
  "object": "card",
  "brand": "visa",
  "last4": "4242",
  "exp_month": 12,
  "exp_year": 2027
}
```

## Masking in logs and interfaces

The most common place sensitive data leaks is not a database breach — it's a log line or a support screen. Apply field-level masking at the boundary, driven by the classification tier rather than ad hoc per screen:

| Field | Stored | Displayed / logged |
|---|---|---|
| Card number | Token only | `•••• •••• •••• 4242` |
| Email | Encrypted | `d••••@example.com` |
| Cardholder name | Encrypted | First name + initial |

!!! note "Why classify before you mask"
    Masking rules are only consistent if they derive from a single classification of each field. When masking decisions are made screen-by-screen, the same field ends up masked in one view and exposed in another. Classify the field once; let every surface inherit the rule. This is the difference between a masking *standard* and a pile of masking *patches*.

## Mapping to your own policy

If your organization maintains its own data classification standard, map Pudgy's tiers onto yours rather than inventing a parallel scheme. The four tiers above align cleanly with the common Restricted / Confidential / Internal / Public model used in most enterprise data-protection programs, which keeps your masking, retention, and access controls consistent across vendors.
