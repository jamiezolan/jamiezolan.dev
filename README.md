# Pudgy Payments — Documentation

A sample developer-documentation site for a fictional payments API, built to demonstrate a modern **docs-as-code** workflow end to end.

> Pudgy does not exist and processes no payments. The point of this repository is the *method*, not the product: how the docs are sourced, written, linted, built, and shipped.

## What this repository demonstrates

| Capability | How it shows up here |
|---|---|
| Docs-as-code | All content is Markdown under Git; the site is generated, never hand-built HTML |
| Static site generator | [MkDocs](https://www.mkdocs.org/) + [Material](https://squidfunk.github.io/mkdocs-material/) |
| Information architecture | Concepts → Guides → Reference split (Diátaxis-influenced) |
| API reference | OpenAPI 3.1 spec rendered as interactive reference |
| Editorial automation | [Vale](https://vale.sh/) prose linting with Microsoft + Google style packages |
| CI/CD | GitHub Actions builds the site and deploys to GitHub Pages on every push to `main` |
| Repo hygiene | Build artifacts are git-ignored; contribution workflow is documented |

## Run it locally

Requires Python 3.9+.

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```

Then open <http://127.0.0.1:8000>. Edits to any Markdown file hot-reload in the browser.

## Build the static site

```bash
mkdocs build          # outputs to ./site (git-ignored)
```

## Lint the prose (optional)

```bash
brew install vale     # macOS
vale sync             # download the style packages referenced in .vale.ini
vale docs/            # lint all Markdown
```

## Deploy

Pushing to `main` triggers `.github/workflows/deploy.yml`, which builds the site and publishes it to GitHub Pages. No manual steps.

## Layout

```
docs/
  index.md                 Landing page
  get-started/             Quickstart + sandbox
  concepts/                Auth, idempotency, errors, pagination, webhooks
  guides/                  Task-based how-tos
  security/                Data classification, PCI scope, tokenization
  reference/               OpenAPI spec + rendered reference
mkdocs.yml                 Site config and navigation
.vale.ini                  Prose linting rules
.github/workflows/         CI build + deploy
```
