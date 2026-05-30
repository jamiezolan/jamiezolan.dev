# Authentication

Pudgy authenticates every request with a secret API key sent as a bearer token. There are no signed query strings and no session cookies — one key, one header.

```bash
curl https://api.pudgypay.dev/v1/charges \
  -H "Authorization: Bearer sk_test_..."
```

A request with a missing, malformed, or revoked key returns `401 Unauthorized` with an `authentication_error`.

## Key types

| Prefix | Environment | Exposure |
|---|---|---|
| `sk_test_` | Sandbox | Server-side only |
| `sk_live_` | Production | Server-side only |
| `pk_test_` / `pk_live_` | Publishable | Safe in client code; can only tokenize cards, never read data |

Secret keys (`sk_`) can do anything your account can do. Treat them like database credentials.

## Handling keys safely

!!! danger "Never expose a secret key"
    A leaked `sk_live_` key lets an attacker move money and read customer records. If a secret key touches a browser, a mobile binary, a public repository, or a log line, rotate it immediately.

Practical rules that hold up in review:

- **Inject keys from the environment**, never hard-code them. Use a secrets manager in production.
- **Use publishable keys in clients.** Card details should be tokenized in the browser with a `pk_` key so a raw card number never reaches your server. See [Tokenization](../security/tokenization.md).
- **Scope keys per service.** Issue a distinct key to each system so you can revoke one without an outage everywhere.
- **Rotate on a schedule and on suspicion.** Pudgy supports overlapping keys so you can rotate with zero downtime: create the new key, deploy it, then revoke the old one.

## Rotating a key without downtime

1. Create a new secret key in the dashboard. Both old and new keys are now valid.
2. Deploy the new key to every service that uses it.
3. Confirm traffic has moved to the new key using the per-key request metrics.
4. Revoke the old key.

Because both keys are live during the overlap, no request fails mid-rotation.
