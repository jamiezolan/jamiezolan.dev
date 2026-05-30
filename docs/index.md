---
hide:
  - navigation
---

<div class="hero" markdown>
# Pudgy Payments

<p class="tagline">A single API to accept payments, manage customers, issue refunds, and react to events in real time — narrated by Pudgy, a hamster who takes correctness, auditability, and clean failure modes very seriously (between naps).</p>
</div>

Pudgy is a sample REST API with a mascot to match. Send a request, get a predictable JSON response, and reconcile every state change through signed webhooks — Pudgy squeaks, you listen. The docs are organized so you can go from your first test charge to a production integration without guessing.

<div class="grid cards" markdown>

-   :material-rocket-launch: **Get started**

    ---

    Create a sandbox key and run your first charge in about five minutes.

    [:octicons-arrow-right-24: Quickstart](get-started/quickstart.md)

-   :material-book-open-variant: **Core concepts**

    ---

    Authentication, idempotency, the error model, pagination, and webhooks — the contracts the whole API rests on.

    [:octicons-arrow-right-24: Authentication](concepts/authentication.md)

-   :material-code-tags: **API reference**

    ---

    Every endpoint, parameter, and response shape, generated from the OpenAPI specification.

    [:octicons-arrow-right-24: Reference](reference/api.md)

-   :material-shield-lock: **Security & compliance**

    ---

    How card data is classified, where PCI scope begins and ends, and why you should never touch a raw PAN.

    [:octicons-arrow-right-24: Data classification](security/data-classification.md)

</div>

## How the API is shaped

Every Pudgy endpoint follows the same conventions, so once you learn one resource the rest are predictable:

- **Base URL.** All requests go to `https://api.pudgypay.dev/v1`.
- **JSON in, JSON out.** Request and response bodies are JSON. Timestamps are ISO 8601 in UTC.
- **Resources are stable.** Objects such as `charge`, `customer`, and `refund` keep the same shape across endpoints.
- **Failures are typed.** Errors return a machine-readable `code` and a human-readable `message`, never a bare HTTP status.
- **Mutations are safe to retry.** Every state-changing request accepts an idempotency key.

!!! note "This is a portfolio sample"
    Pudgy is a fictional service used to demonstrate a modern docs-as-code workflow: Markdown source under version control, an OpenAPI-driven reference, prose linting in CI, and automated deploys. No real payments are processed, no endpoint resolves, and no hamsters were overfed in the making of these docs.
