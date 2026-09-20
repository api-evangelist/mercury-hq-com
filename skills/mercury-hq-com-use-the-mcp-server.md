---
generated: '2026-09-19'
method: generated
name: Call MERCURY through its MCP server (hosted with an API key, or stdio with a wallet)
description: Discover the 18 tools for free, then call them either on the hosted streamable-HTTP endpoint with a Mercury API key or locally through npx mercury-x402-mcp paying from your own wallet.
api: mcp/mercury-hq-com-mcp.yml
operations: [buy_web_fetch, buy_cited_robots, buy_cited_sitemap, buy_cited_dns]
source: >-
  Live initialize + tools/list on https://network.mercury-hq.com/mcp (2026-09-19); GET /mcp descriptor;
  npm README of mercury-x402-mcp 1.0.1; key issuance from /university/developers; tool-to-operation binding
  from mcp/mercury-hq-com-tool-crosswalk.yml.
---

# Call MERCURY through its MCP server

Two deployments of one tool catalog. The hosted server is reachable by any MCP client right now; the stdio package needs a machine and a funded wallet.

## Hosted (remote) — `POST https://network.mercury-hq.com/mcp`
1. `initialize` (protocolVersion `2025-06-18`) and `tools/list` are free and unauthenticated: 18 tools — `fetch, extract, markdown, metadata, links, robots, diff, notarize, headers, table, feed, availability, validate, batch, sitemap, dns, readability, redirect` — each with a JSON Schema `inputSchema` identical to the query parameters of its `/buy/*` operation.
2. Get a key: `POST https://network.mercury-hq.com/api/dev/keys?live=false` → `201 {key: "mk_test_…", …}` (shown once; 100 credits; 5/min; 20 keys per IP per hour). Send it as `Authorization: Bearer mk_test_…` on `tools/call`.
3. Call, e.g. `tools/call {name: "robots", arguments: {domain: "example.com"}}` (backs `buy_cited_robots`, 5 credits = $0.005): a signed per-AI-crawler allow/block audit from robots.txt, llms.txt and ai.txt. `sitemap` (`buy_cited_sitemap`, `url`, optional `fetch` 0–10 and `limit`) and `dns` (`buy_cited_dns`, `domain` or `url`) plan a crawl the same way.
4. Tool names are bare verbs: `web_fetch` returns `isError:true "Unknown tool"` — use `fetch` (backs `buy_web_fetch`).

## Local (stdio) — `npx -y mercury-x402-mcp`
1. MCP host config: `{"mcpServers":{"mercury":{"command":"npx","args":["-y","mercury-x402-mcp"],"env":{"MERCURY_PRIVATE_KEY":"<Base mainnet wallet key>"}}}}`.
2. The server builds its tool list live from `/catalog` and pays each call over x402 from that wallet (no Mercury key). `mercury_catalog` and `mercury_verify` work with no wallet.
3. The package was last published 2026-06-07 (17 tools then; 18 live now) — expect the live list, not the README's.

## Errors and limits
- No OAuth and no RFC 9728 metadata on the MCP host: a client cannot register dynamically; keys come from the anonymous `POST /api/dev/keys`.
- Every successful `tools/call` on either deployment is a purchase: no idempotency, no refunds (`conventions/mercury-hq-com-conventions.yml`). See `errors/mercury-hq-com-problem-types.yml`.
