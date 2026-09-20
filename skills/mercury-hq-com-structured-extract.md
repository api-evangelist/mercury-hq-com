---
generated: '2026-09-19'
method: generated
name: Turn a page into a typed JSON record and validate an API response
description: Use the Extract faculty - schema-driven extraction, signed metadata, HTML tables and JSON Schema validation - to get records an agent can act on, each bound to a signed receipt.
api: openapi/mercury-hq-com-x402-storefront-openapi.yml
operations: [buy_extract, buy_cited_metadata, buy_cited_table, buy_cited_validate]
source: >-
  operationIds and parameter constraints verified in openapi/mercury-hq-com-x402-storefront-openapi.yml;
  schema syntax from the live MCP tools/list inputSchema descriptions (mcp/mercury-hq-com-mcp-tools-list.json);
  prices from /catalog (2026-09-19).
---

# Turn a page into a typed JSON record and validate an API response

All four operations are `GET`s paid per call over x402 (or with a `Bearer mk_` key) and return `{ok, url, status, data|extract, fetchedAt, attestation}`. They are deterministic and use no LLM: the same source bytes give the same record, which is what makes the signed receipt meaningful.

## Auth
- 402 → pay → replay, or `Authorization: Bearer mk_test_...`. See `authentication/mercury-hq-com-authentication.yml`.

## Steps
1. **Schema extract** — `buy_extract` (`GET /buy/extract`, $0.004) with `url` and `schema` (both required). `schema` is either a compact list `field[:type]` such as `title,price:number,rating:number,inStock:boolean` (types string|number|integer|boolean, default string) or a JSON Schema string (≤1024 chars). Response: `extract{}` typed per the schema, `schema` echoed, `coerced` flags, plus `title`, `text`, `redirects`, `metered`.
2. **Metadata record** — `buy_cited_metadata` (`GET /buy/metadata?url=`, $0.006) returns one signed record built from JSON-LD, OpenGraph, Twitter card, `<meta>`, canonical and title under `data`.
3. **Tables** — `buy_cited_table` (`GET /buy/table?url=`, $0.006) returns the page's HTML tables as header-keyed JSON rows and RFC 4180 CSV; `table=<0-based index>` selects one, `format=both|json|csv`.
4. **Validate a JSON API** — `buy_cited_validate` (`GET /buy/validate`, $0.005) with `url` (a JSON endpoint) and `schema` (required, ≤16384 chars; compact list or JSON Schema); optional `pointer` (JSON Pointer to the sub-document). Returns a pass/fail verdict with per-field errors, signed over the exact response bytes.
5. **Keep the receipt with the record** — the attestation covers the extracted fields, so a downstream agent can prove the record was resolved from that URL at that time without trusting you.

## Errors
- `ok:false` on 200 is a charged, honest failure — check it before parsing `data`. Parameter constraints (`maxLength`, required `schema`) are enforced but the 400 shape is unpublished. See `errors/mercury-hq-com-problem-types.yml`.

## Notes
- No pagination, no idempotency key; each retry is a new charge (`conventions/mercury-hq-com-conventions.yml`).
- MCP equivalents: tools `extract`, `metadata`, `table`, `validate` (`mcp/mercury-hq-com-tool-crosswalk.yml`).
