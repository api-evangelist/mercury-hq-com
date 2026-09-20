---
generated: '2026-09-19'
method: generated
name: Fetch a page with a verifiable provenance receipt over x402
description: Pay $0.003 USDC per call over HTTP 402 to read a public URL and receive clean text plus an EIP-191 signed receipt you can verify offline; optionally take markdown or an article record instead.
api: openapi/mercury-hq-com-x402-storefront-openapi.yml
operations: [buy_web_fetch, buy_cited_markdown, buy_cited_readability]
source: >-
  operationIds verified in openapi/mercury-hq-com-x402-storefront-openapi.yml; the 402 body observed live on
  https://network.mercury-hq.com/buy/fetch on 2026-09-19; prices from /catalog; verification recipe from
  /.well-known/mercury-attestation; refund policy from /terms.
---

# Fetch a page with a verifiable provenance receipt over x402

MERCURY is keyless: there is no signup and no API key on this rail. Your agent needs a Base-mainnet wallet holding a little USDC and an x402 client. Every call is a purchase (from $0.003), charged per attempt and non-refundable, so decide before you call, not after.

## Auth
- None up front. The first request is refused with **402** and an x402 v1 challenge; replaying it with a signed `X-PAYMENT` header succeeds. `x402-fetch` (`wrapFetchWithPayment(fetch, account)` with a viem account) does the loop for you. See `authentication/mercury-hq-com-authentication.yml`.
- Alternative: `Authorization: Bearer mk_test_...` (free, 100 credits, from `https://network.mercury-hq.com/developers`) on the same routes.

## Steps
1. **Rehearse for free** — `GET https://network.mercury-hq.com/buy/fetch?url=https://example.com` with no payment. Expect **402** with `accepts[0].maxAmountRequired = "3000"` (USDC base units = $0.003), `payTo`, `asset 0x8335...2913`, `maxTimeoutSeconds 60` and an `outputSchema` for the result. Nothing is charged.
2. **Fetch** — `buy_web_fetch` (`GET /buy/fetch`) with `url` (required, ≤2048 chars). Options: `format=markdown` (structure-preserving markdown), `links=1` (adds `links[]`, same-origin first), `extract=1` (adds `description` + `wordCount`) or `extract=title,price,author,publishedAt` (adds a typed `extract{}` record). The add-ons move you to the `plus` ($0.006) or `pro` ($0.012) tier declared in the operation's `x-x402.accepts`.
3. **Read `ok` first** — a **200** with `ok:false` and `error` is an *honest failure* (target down, SSRF-blocked, >5 s, >10 MB): delivered and charged. Do not retry in a loop; each settled retry is a new charge (`conventions/mercury-hq-com-conventions.yml`, idempotency `coverage: none`).
4. **Verify the receipt offline** — from `attestation`: recompute `sha256(text)` and compare with `attestation.contentHash`; rebuild `attestation.verify.message` (template `mercury-x402:fetch-attestation:v1\nurl=…\nstatus=…\nsha256=…\nfetchedAt=…\nnonce=…`); EIP-191 `ecrecover` the `signature` and require the signer `0xACB40253BD71Bb9a5d491b2c6EFF755F2A33Fc75` (pinned at `/.well-known/mercury-attestation`). No call back to MERCURY is needed; `POST /verify {text, attestation}` is a free second opinion.
5. **Prefer a different shape?** `buy_cited_markdown` (`GET /buy/markdown?url=`, $0.005) returns LLM-ready markdown; `buy_cited_readability` (`GET /buy/readability?url=`, $0.005) returns an article record `{title, byline, publishedAt, text}`. Same 402 flow, same receipt.

## Errors
- `402` is the designed first answer. `404 {ok:false, error:"No x402 resource at …"}` means a wrong route — read `/catalog`. `503 {reason:"mint-disabled"}` (Retry-After 3600) is a gated SKU and is never charged. See `errors/mercury-hq-com-problem-types.yml`.

## Notes
- Prices: `plans/mercury-hq-com-plans-pricing.yml`. No rate limit is published on the x402 rail; the key rail is 5/min on the free tier (`rate-limits/mercury-hq-com-rate-limits.yml`).
- The same page can be fetched through the MCP tool `fetch` or the A2A skill `web-fetch` (free preview via `message/send` on `/a2a`).
