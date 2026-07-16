<!--
WHY NOW: pairs with promo/devto/sec-edgar-free-json-api.md (seed #2 in promo/PIPELINE.md,
actor sec-edgar-scraper) — completes the promo pair (article + cheatsheet) the way
noble-ronin/ats-job-apis paired with the job-boards dev.to piece. No new topic research
needed: same endpoints, reference format instead of narrative.

VERIFICATION LOG (Rule #0): same three endpoints as the paired dev.to article, whose log
already cross-checked SEC's own developer-resources / edgar/sec-api-documentation pages
against 3 independent technical write-ups (tldrfiling.com ×2, sec-edgar-api.readthedocs.io).
CAVEAT carried over: this session's sandboxed network policy again blocked live fetch —
tried WebFetch on data.sec.gov, www.sec.gov, and a control fetch of example.com; all three
returned 403 at the proxy level (recentRelayFailures: connect_rejected), confirming it's a
sandbox policy, not an SEC-specific block. No new endpoint added beyond what the article
already verified via documentation, so nothing here rests on unverified ground — but same
suggestion stands: sanity-check one URL in a browser before publishing.
-->

# SEC EDGAR JSON API — a cheatsheet

![SEC Filings Have a Free JSON API](assets/banner-1.png)

The SEC publishes every filing — 10-Ks, 8-Ks, insider trades (Form 4), 13F holdings,
S-1s — through a free, keyless JSON API. No sign-up, no API key, no paid tier. The only
requirement is a descriptive `User-Agent` header; the only limit is a flat rate cap.

## Endpoints

| Endpoint | Purpose | Auth | Notes |
|---|---|---|---|
| `GET https://www.sec.gov/files/company_tickers.json` | Ticker → CIK lookup for every public company | None (User-Agent required) | Keyed by row index, not ticker; fields: `cik_str`, `ticker`, `title` |
| `GET https://data.sec.gov/submissions/CIK##########.json` | Full filing history + company metadata | None (User-Agent required) | CIK zero-padded to 10 digits; returns `filings.recent` (form, filingDate, reportDate, accessionNumber) |
| `GET https://efts.sec.gov/LATEST/search-index?q=&forms=&startdt=&enddt=` | Full-text search across every filing | None (User-Agent required) | Elasticsearch-style response under `hits.hits`; params: `q`, `forms`, `startdt`/`enddt` |

## Examples

```bash
# Ticker -> CIK (Apple = 320193)
curl -s -H "User-Agent: Your Name your@email.com" \
  https://www.sec.gov/files/company_tickers.json | head -c 200

# Full filing history for a CIK
curl -s -H "User-Agent: Your Name your@email.com" \
  https://data.sec.gov/submissions/CIK0000320193.json | head -c 400

# Full-text search, scoped to 10-Ks in a date range
curl -s -H "User-Agent: Your Name your@email.com" \
  "https://efts.sec.gov/LATEST/search-index?q=%22material+weakness%22&forms=10-K&startdt=2026-01-01&enddt=2026-06-30"
```

A `filings.recent` entry (submissions endpoint) looks like:

```json
{
  "form": "10-K",
  "filingDate": "2025-11-01",
  "reportDate": "2025-09-27",
  "accessionNumber": "0000320193-25-000123"
}
```

Turn an `accessionNumber` into a document URL by stripping the dashes and joining it
under `https://www.sec.gov/Archives/edgar/data/<CIK>/<accession-no-dashes>/`.

## Gotchas

- **Send a real `User-Agent`.** An actual name + contact email, not a browser string —
  that's SEC's entire access control, and it's enforced.
- **10 requests/second**, combined across `data.sec.gov` and `efts.sec.gov`. Fine for
  normal use; don't hammer it in a tight loop.
- CIKs must be **zero-padded to 10 digits** in the submissions URL (`320193` →
  `0000320193`) — the ticker file gives you the unpadded number.
- `company_tickers.json` is keyed by row index (`"0"`, `"1"`, …), not by ticker — you
  have to scan values, not index directly by symbol.

## More

Same free official API underneath, wrapped so you skip the glue code (CIK padding,
form-type filtering, pagination, rate-limit backoff, accession → URL): [SEC EDGAR
Filings Scraper & Full-Text
Search](https://apify.com/ponderable_hydrometer/sec-edgar-scraper).

Full write-up with more context: "SEC Filings Have a Free JSON API" on
[dev.to/ronin13](https://dev.to/ronin13) (link goes live once that article is published).
