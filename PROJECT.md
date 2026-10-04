# Northstar Search Intelligence — Project Brief

## POV

Retail search often fails before retrieval: customers describe a need, while catalogs describe products. The core hypothesis is that a small, catalog-specialized Liquid LFM can close that language gap quickly and efficiently.

One domain model should improve both:

- search, by expanding or rewriting a customer's words into catalog language;
- the assistant, by grounding answers, comparisons, and recommendations in the same product truth.

The demo should make that improvement visible without hiding it behind a large orchestration layer. The harness routes requests, validates structured output, retrieves products, and renders results. Product understanding belongs in the model.

## Demo promise

A customer can search or ask naturally—for example, “light shoes for standing all day,” “something for a rainy commute,” or “compare the hoodies”—and receive relevant, catalog-grounded products and explanations.

The experience is deliberately small enough to understand end to end:

- 24 synthetic products across hoodies, T-shirts, sneakers, slides, socks, backpacks, wallets, and fragrance;
- structured attributes, variants, descriptions, tags, prices, and unique imagery;
- two model touchpoints: the search bar and the floating shopping assistant;
- cart and single-page demo checkout to complete the retail journey.

## Current state

Implemented:

- responsive minimalist storefront;
- 24 products with unique imagery;
- category browsing and product-detail dialogs;
- colors, sizes, materials, tags, badges, and prices;
- deterministic lexical filtering in the search bar;
- scripted assistant responses for a few representative intents;
- cart, quantity management, and demo checkout;
- `search_catalog` and `add_product_to_cart` browser tool registrations;
- private hosted deployment.

Not yet implemented:

- external model inference;
- query rewriting or semantic retrieval;
- model-grounded assistant answers;
- a standalone catalog corpus or training dataset;
- evaluation and latency instrumentation;
- Liquid LFM fine-tuning or on-device inference.

The UI labels scripted chat replies as prototype responses so the demo does not imply that model integration already exists.

## Product data

The current catalog is embedded in `app/page.tsx`. Each product contains:

| Field | Purpose |
| --- | --- |
| `id` | Stable retrieval and cart identifier |
| `name`, `category` | Primary merchandising identity |
| `price` | Transactional display value |
| `description` | Customer-facing product summary |
| `material` | Searchable factual attribute |
| `colors`, `sizes` | Product variants |
| `tags` | Lightweight intent and synonym coverage |
| `image`, `position` | Catalog presentation |
| `badge` | Optional merchandising label |

Before model work, move this data into a versioned JSON or JSONL source of truth. UI code should consume the same records used for retrieval, training examples, and evaluation.

## Corpus plan

Keep the first corpus compact and inspectable. It should contain five layers:

1. **Product truth** — one normalized record per product and variant, including only facts the assistant may state.
2. **Retail vocabulary** — synonyms, category aliases, materials, use cases, and attribute mappings such as “trainers” → sneakers or “standing all day” → cushioning and comfort.
3. **Query-to-product examples** — terse searches, natural-language needs, misspellings, ambiguous queries, and expected product IDs or filters.
4. **Assistant examples** — grounded product questions, comparisons, recommendations, and safe “not in catalog” responses.
5. **Hard negatives** — plausible but incorrect products and unsupported claims that teach boundaries.

Suggested initial scale:

- 24 product records;
- 10–20 query variants per product or intent cluster;
- 50–100 comparison and product-QA examples;
- explicit negative examples for unsupported inventory, attributes, and policies.

Synthetic examples are acceptable for the POC, but every generated example must resolve to catalog facts and stable product IDs.

## Target inference design

### Search touchpoint

```text
customer query
  → model returns normalized intent, expansions, and optional filters
  → deterministic catalog retrieval/ranking
  → product grid plus a visible interpretation of the query
```

The model should return structured data, not final HTML. A minimal contract could include:

```json
{
  "normalized_query": "cushioned walking shoes",
  "expansions": ["daily trainer", "standing comfort"],
  "category": "Sneakers",
  "attributes": ["cushioned", "walking", "comfortable"]
}
```

### Assistant touchpoint

```text
customer message
  → intent/query interpretation
  → retrieve a small set of catalog records
  → model answers only from supplied records
  → optional product-card or add-to-cart action
```

Both paths share catalog normalization and retrieval. They should not maintain separate product knowledge.

## Model strategy

The integration should be provider-agnostic at first:

- use a small frontier model through a provider such as OpenRouter to validate prompts, contracts, and UX quickly;
- capture representative inputs and expected structured outputs as the seed evaluation set;
- replace the interpretation and answer-generation stages with a Liquid LFM tiny model;
- specialize only after the corpus and failure modes are stable.

This is not a baseline bake-off. The frontier model is an integration scaffold, while the demo's destination is a compact Liquid model specialized on the retail domain.

## Thin-harness rules

- Keep retrieval and filtering deterministic and observable.
- Ask the model for narrow structured outputs.
- Do not encode every synonym or demo answer in application conditionals.
- Do not let the harness silently repair weak model outputs beyond schema validation and safe fallback.
- Log the original query, model interpretation, retrieved product IDs, answer, latency, and fallback reason.
- Keep one provider interface so model changes do not require UI rewrites.

## Demo success criteria

The POC is successful when it can reliably demonstrate:

- natural need-based queries finding sensible products even without exact keyword overlap;
- assistant answers that cite only available product facts;
- useful comparisons between two or more catalog items;
- graceful handling of ambiguous or out-of-catalog requests;
- a visibly small orchestration layer;
- responsive inference suitable for interactive search and chat.

For the first pass, use a curated demo/evaluation set rather than building a full benchmark dashboard. Record relevance, groundedness, structured-output validity, fallback rate, and end-to-end latency.

## Scope boundaries

In scope:

- catalog construction and normalization;
- search interpretation and expansion;
- lightweight retrieval and ranking;
- grounded product assistance;
- model swapping and focused evaluation.

Out of scope for this POC:

- production inventory, accounts, payments, fulfillment, or analytics;
- personalization and long-term memory;
- a large vector platform or agent framework;
- broad web knowledge;
- production security and compliance certification.

## Delivery phases

1. **Storefront foundation — complete.** Catalog, imagery, browsing, cart, checkout, search surface, assistant surface, and hosting.
2. **Data extraction.** Move products to JSON/JSONL; add aliases, use cases, facts, and evaluation queries.
3. **Provider seam.** Add one server-side model adapter and validated structured contracts.
4. **Search intelligence.** Connect query expansion to deterministic retrieval and expose the interpretation in the UI.
5. **Grounded assistant.** Retrieve products, answer from product truth, and support product/cart actions.
6. **Evaluation.** Run the curated suite and capture quality, validity, fallback, and latency.
7. **Liquid specialization.** Swap in and, if useful, fine-tune a Liquid LFM tiny model using the stabilized corpus.

Any expansion beyond these phases should be justified by a demo failure, not by general platform ambition.
