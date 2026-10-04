# Northstar Search Intelligence

Northstar is a compact retail proof of concept for demonstrating how domain-specialized Liquid LFM models can improve two customer touchpoints backed by the same product catalog:

1. onsite product search and query expansion;
2. a grounded shopping assistant for discovery, comparison, and product questions.

The current release is a polished, deployable storefront with a 24-product synthetic catalog, product variants, cart, one-page checkout, lexical search, and a scripted assistant. Model inference is intentionally the next integration step.

## Documentation

- [PROJECT.md](./PROJECT.md) — product POV, scope, data strategy, target architecture, and roadmap
- [RUNBOOK.md](./RUNBOOK.md) — local setup, demo script, verification, deployment, and integration seams

## Quick start

Requires Node.js 22.13 or newer.

```sh
npm ci
npm run dev
```

Open <http://localhost:5173>.

```sh
npm run build
npm run lint
```

## Current deployment

<https://northstar-search-intelligence.liquid-ai-in-2006.chatgpt.site>

The hosted Site is private. The storefront and checkout are demonstrations only; no payment is processed.

## Repository scope

The repository is intentionally centered on the POC:

- `app/` — Northstar storefront, catalog, search, assistant, cart, and checkout
- `public/catalog/products/` — the 24 product images used by the demo
- `components/ui/` — only the three UI primitives used by the storefront
- `PROJECT.md` and `RUNBOOK.md` — product POV and operating plan

The smaller `build/`, `scripts/`, `lib/`, `vendor/`, and configuration surfaces are retained because Vinext and Sites use them to preview, build, and publish the application. Generated dependencies and runtime state—including `node_modules`, `dist`, `.vite`, `.vinext`, `.wrangler`, and local environment files—are excluded by `.gitignore`.

## License

Released under the [MIT License](./LICENSE).
