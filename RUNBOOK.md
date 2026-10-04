# Northstar Demo Runbook

## 1. Purpose

Use this runbook to start, verify, present, and extend the Northstar retail search-intelligence POC. For product rationale and architecture, read [PROJECT.md](./PROJECT.md).

## 2. Prerequisites

- Node.js 22.13 or newer
- npm
- Git only when publishing

The current app has no database or required runtime secrets. Catalog data is local and checkout is simulated.

## 3. Local setup

From the `storefront` directory:

```sh
npm ci
npm run dev
```

Open <http://localhost:5173>.

The dev server uses Vite/Vinext hot reload. Avoid starting a second server on the same port.

## 4. Verification

Before a demo or deployment:

```sh
npm run build
npm run lint
```

Manual smoke test:

1. Confirm the page shows 24 products and distinct product imagery.
2. Select each major category and confirm the grid updates.
3. Search `hoodie`, `water resistant`, and `blue`; confirm matching products remain.
4. Search an impossible phrase; confirm the empty state appears and can be cleared.
5. Open a product, change its variant, and add it to the bag.
6. Change cart quantity and confirm totals update.
7. Complete the demo checkout and confirm no real payment is attempted.
8. Open **Ask Northstar** and try all three suggested prompts.
9. Check mobile and desktop widths.

## 5. Current demo flow

Recommended five-minute walkthrough:

1. **Set the problem.** Retailers describe inventory with structured attributes; customers describe outcomes and needs.
2. **Show the catalog.** Browse the eight categories and open a product to show variants and product facts.
3. **Show today's search surface.** Search by a known word or attribute. State clearly that current matching is deterministic lexical filtering.
4. **Show the intended intelligence gap.** Use a need-based query that lexical matching handles poorly. Explain that query interpretation and expansion will sit here.
5. **Open the assistant.** Ask for a weekend backpack, a hoodie comparison, and products for rain. State clearly that current replies are scripted placeholders.
6. **Connect the POV.** The same catalog-specialized model will power query interpretation and grounded assistant responses.
7. **Complete the journey.** Add a recommended item to the bag and show the one-page demo checkout.

Do not present current search or chat behavior as Liquid model inference. The model connection is the next project phase.

## 6. Current implementation map

| Area | Location | Current behavior |
| --- | --- | --- |
| Catalog and UI | `app/page.tsx` | 24 in-memory product records and storefront interactions |
| Search | `app/page.tsx` `filtered` memo | Case-insensitive all-term lexical matching |
| Assistant | `app/page.tsx` `sendMessage` | Scripted intent branches with a short delay |
| Browser tools | `app/page.tsx` `registerTool` | Catalog search and add-to-cart actions |
| Styling | `app/globals.css` | Tailwind-based application styling |
| Product media | `public/catalog/products/` | One optimized image per product |
| Hosting | `.openai/hosting.json` | Existing private Sites project |

The tracked `build/`, `scripts/`, `lib/`, and `vendor/` files are the minimal Vinext/Sites infrastructure required for preview and deployment. Unused starter examples and UI components are intentionally excluded.

## 7. Model-integration seam

Do not call a model directly from the browser or expose provider keys in client code. Add server-side endpoints or actions behind a small provider interface.

Recommended logical interface:

```ts
type QueryInterpretation = {
  normalizedQuery: string;
  expansions: string[];
  category?: string;
  attributes: string[];
};

interface RetailModel {
  interpretQuery(query: string): Promise<QueryInterpretation>;
  answerWithCatalog(input: {
    message: string;
    products: ProductRecord[];
  }): Promise<{ answer: string; productIds: string[] }>;
}
```

Planned server-side configuration—not implemented yet:

```text
RETAIL_MODEL_PROVIDER=openrouter|liquid
RETAIL_MODEL_ID=<provider model identifier>
OPENROUTER_API_KEY=<server secret when OpenRouter is used>
```

Validate every model response against a schema. On timeout, malformed output, or provider failure, fall back to literal catalog search and a transparent assistant error. Do not fabricate product facts.

## 8. Data preparation

Before connecting inference:

1. Extract the 24 products from `app/page.tsx` into a versioned catalog file.
2. Preserve stable product IDs.
3. Add normalized aliases, use cases, and factual attributes.
4. Create query-to-product examples with expected IDs and filters.
5. Create assistant QA and comparison examples grounded in catalog records.
6. Add ambiguous, unsupported, and hard-negative cases.
7. Split examples into development and held-out evaluation sets before specialization.

Suggested layout:

```text
data/
  catalog.json
  vocabulary.json
  training/
    search.jsonl
    assistant.jsonl
  eval/
    search.jsonl
    assistant.jsonl
```

## 9. Evaluation loop

For each curated case, capture:

- original input;
- structured model interpretation;
- retrieved product IDs;
- expected product IDs or allowed answer facts;
- schema validity;
- groundedness/relevance judgment;
- fallback reason, if any;
- model and end-to-end latency.

Keep the first evaluation runner simple and command-line based. A dashboard is unnecessary until repeated experiments justify it.

## 10. Build and publish

Build locally first:

```sh
npm run build
```

The deployed Site is:

<https://northstar-search-intelligence.liquid-ai-in-2006.chatgpt.site>

Publishing should update the existing Sites project declared in `.openai/hosting.json`; do not create a replacement project. Preserve its current private access unless sharing requirements change explicitly.

After publishing, verify the returned production deployment succeeded and rerun the manual smoke test on the live URL.

## 11. Troubleshooting

### Port 5173 is already in use

Use the existing preview or stop the stale process. To use another port:

```sh
npm run dev -- --port 5174
```

### Product image does not load

Confirm the product's `image` value points to an existing file under `public/catalog/products/` and matches filename case exactly.

### Search returns nothing unexpectedly

Current search requires every entered term to occur somewhere in the concatenated product fields. This is a known limitation and the reason for the planned query-expansion stage.

### Assistant repeats a generic recommendation

That is expected until model integration. The current assistant recognizes only weekend/trip, rain/water, and compare/hoodie branches.

### Hosted behavior differs from localhost

Run `npm run build`, confirm browser-only APIs are guarded, and verify that no secret or model call is being made from client code.

## 12. Documentation maintenance

Update these files whenever behavior changes:

- `README.md` for setup, links, and current status;
- `PROJECT.md` for scope, POV, architecture, corpus strategy, and roadmap;
- `RUNBOOK.md` for commands, configuration, demo steps, and operational checks.

When model inference is added, remove the placeholder warnings only after both search and assistant are actually connected and verified.
