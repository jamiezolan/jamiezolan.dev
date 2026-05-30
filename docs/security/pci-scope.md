# PCI DSS scope

PCI DSS is the security standard that applies to anyone who stores, processes, or transmits cardholder data. The single most effective way to reduce your compliance burden is to reduce your *scope* — the set of systems that touch card data. Pudgy is built to keep that set as small as possible.

## Scope is about where the card number goes

Any system that handles a raw card number is "in scope" and must meet the full set of PCI controls. The goal of a well-architected integration is to ensure that **no raw card number ever reaches your systems**.

| Integration style | Where the PAN goes | Your PCI scope |
|---|---|---|
| Hosted fields / tokenization in the browser | Browser → Pudgy directly | Smallest (SAQ A-style) |
| Card data posts to your server, then to Pudgy | Through your servers | Largest — full controls apply |

The first row is the target. The card number travels from the customer's browser straight to Pudgy and comes back as a token; your servers only ever see the token.

## How Pudgy keeps you out of scope

- **Tokenization at the edge.** Card details are captured by a Pudgy-hosted field using your publishable key, so the PAN bypasses your server entirely. See [Tokenization](tokenization.md).
- **No PAN in responses.** The API never returns a full card number, so you can't accidentally log or store one.
- **No CVV storage.** CVV is used to authorize and then discarded; storing it is prohibited and Pudgy gives you no way to do it.

## What you're still responsible for

Reducing scope is not eliminating responsibility. You remain accountable for:

- **Protecting your API keys.** A leaked secret key is a security incident regardless of PCI scope — see [Authentication](../concepts/authentication.md).
- **Securing your integration code.** TLS everywhere, dependency hygiene, and access control on the systems that call Pudgy.
- **Completing the right self-assessment.** Even a minimal integration requires the appropriate SAQ. Confirm which one applies with your acquirer or QSA.

!!! warning "This page is guidance, not an attestation"
    Scope determination is specific to your architecture and must be validated with a Qualified Security Assessor or your acquiring bank. Nothing here substitutes for that.
