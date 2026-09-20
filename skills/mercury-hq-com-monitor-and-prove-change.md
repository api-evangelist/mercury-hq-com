---
generated: '2026-09-19'
method: generated
name: Monitor a URL and prove it changed (availability, diff, notarize, batch)
description: Build a tamper-evident audit trail - probe availability, detect and prove a content change against a prior receipt, notarize arbitrary bytes, and snapshot up to 20 pages under one Merkle root.
api: openapi/mercury-hq-com-x402-storefront-openapi.yml
operations: [buy_cited_availability, buy_cited_diff, buy_cited_notarize, buy_cited_batch]
source: >-
  operationIds and parameters verified in openapi/mercury-hq-com-x402-storefront-openapi.yml; response
  fields from the inline 200 schemas; prices from /catalog (2026-09-19).
---

# Monitor a URL and prove it changed

The Verify and Monitor faculties turn "it was up / it changed / it said this at time T" into signed evidence. Receipts chain: the `contentHash` of one fetch is the `prevHash` of the next diff.

## Auth
- 402 → pay → replay, or `Authorization: Bearer mk_test_...`. See `authentication/mercury-hq-com-authentication.yml`.

## Steps
1. **Baseline** — fetch once with `buy_web_fetch` (or `buy_cited_notarize`, below) and keep `attestation.contentHash`.
2. **Probe availability** — `buy_cited_availability` (`GET /buy/availability?url=`, $0.005): up/down, HTTP status and class, reachability reason, final URL after redirects, measured `responseMs`, signed.
3. **Prove a change** — `buy_cited_diff` (`GET /buy/diff`, $0.006) with `url` and either `prevHash` (0x + 64 hex, from the baseline receipt; hash-only change proof) or `prevText` (≤256 KB; produces a textual diff); optional `format=text|markdown`. Response: `changed`, `fromHash`, `toHash`, `comparedBy`, `diff`, `text`, `attestation` binding fromHash→toHash.
4. **Notarize bytes you already hold** — `buy_cited_notarize` (`GET /buy/notarize`, $0.008) with exactly one of `url` (fetch and notarize the cleaned text) or `content` (inline UTF-8 ≤256 KB). Returns a signed receipt with `contentHash` and witnessed timestamp.
5. **Snapshot a set** — `buy_cited_batch` (`GET /buy/batch`, $0.02) with `urls` (comma- or newline-separated, ≤20, deduped) and optional `format`. One receipt commits to a Merkle root over every page's `contentHash` with per-page membership proofs.
6. **Store the receipts, not the trust** — each receipt is verifiable offline forever against the pinned key (`/.well-known/mercury-attestation`).

## Errors
- Every route may return 200 `ok:false` for an unreachable target — that is a charged attempt, and for availability it is the *result*. See `errors/mercury-hq-com-problem-types.yml`.

## Notes
- No schedule or webhook exists on MERCURY's side: the agent runs the loop and pays per probe (`capabilities.pushNotifications` is false on the A2A card; no AsyncAPI/webhooks published).
- MCP equivalents: tools `availability`, `diff`, `notarize`, `batch`.
