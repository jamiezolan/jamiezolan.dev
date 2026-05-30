# Contributing

This site is docs-as-code: documentation is source, changes go through pull requests, and CI checks every change before it ships. Writers and engineers use the same workflow.

## Workflow

1. **Branch.** Create a branch from `main`: `git checkout -b docs/clarify-idempotency`.
2. **Edit.** Change Markdown in `docs/`. Run `mkdocs serve` to preview with hot reload.
3. **Lint.** Run `vale docs/` and resolve issues before opening the PR.
4. **Open a PR.** CI builds the site and runs the prose linter. A failing build blocks the merge.
5. **Review and merge.** On merge to `main`, the site deploys automatically.

## Style

- Address the reader as **you**. Prefer the active voice.
- Lead with the task, then the detail. A reader scanning headings should find what they need without reading prose.
- Show, then explain. Every concept page earns its keep with a runnable example.
- Keep one source of truth. The API reference is generated from `openapi.yaml`; never hand-edit endpoint details into prose.

Prose style is enforced mechanically by [Vale](https://vale.sh/) using the Microsoft and Google style packages — see `.vale.ini`. This keeps reviews focused on substance instead of comma placement.

## Information architecture

Pages are organized along the [Diátaxis](https://diataxis.fr/) split so each one has a single job:

| Section | Answers | Voice |
|---|---|---|
| Concepts | "How does this work?" | Explanatory |
| Guides | "How do I accomplish X?" | Task-oriented |
| Reference | "What exactly does this field do?" | Precise, generated |

When you add a page, decide which job it does. A page that tries to do all three does none well.
