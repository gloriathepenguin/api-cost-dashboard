# API Cost Reference

Third-party API calls across all brand-template skills.
Maintained as the single source of truth for cost estimation and budget planning.

## Official API References

| Provider | Docs URL |
|----------|----------|
| Ahrefs | https://docs.ahrefs.com/docs/api/reference/introduction |
| AppTweak | https://developers.apptweak.com/reference/overview |
| Firecrawl | https://docs.firecrawl.dev/api-reference/v2-introduction |
| DataForSEO | https://docs.dataforseo.com/v3/ |
| Deepgram | https://developers.deepgram.com/reference/deepgram-api-overview |
| ElevenLabs | https://elevenlabs.io/docs/api-reference/introduction |
| ScreenshotOne | https://screenshotone.com/docs/getting-started/ |
| fal.ai (Seedance 2.0) | https://fal.ai/docs |
| Creatify | https://docs.creatify.ai/api-reference/introduction |
| Foreplay | https://foreplay.co/api |
| Google PageSpeed | https://developers.google.com/speed/docs/insights/v5/reference/pagespeedapi/runpagespeed |

---

## Subscription Plans

| Provider | Plan | Monthly cost | Quota | Unit rate |
|----------|------|-------------|-------|-----------|
| AppTweak | Small | $199/mo | 250,000 credits | $0.000796/cr |
| Ahrefs | Standard | $249/mo | 400,000 units | $0.0006225/unit |
| Foreplay | Pro | $99/mo | 100,000 credits | $0.00099/cr |
| Creatify | Starter | $99/mo | 500 credits | $0.198/cr |
| Deepgram | Pay-as-you-go | Usage-based | — | $0.0059/min (Nova-2) |
| ElevenLabs | Pay-as-you-go | Usage-based | — | ~$0.30/1K chars (TBC) |
| Firecrawl | Starter | Usage-based | — | ~$0.002/page |
| ScreenshotOne | Pay-as-you-go | Usage-based | — | ~$0.003/screenshot |
| DataForSEO | Pay-as-you-go | Usage-based | — | Labs: $0.01/task + $0.0001/item · SERP Live: $0.002/query |
| fal.ai | Pay-as-you-go | Usage-based | — | $0.30/s std · $0.24/s fast (Seedance) |

**Total fixed subscriptions: $646/mo** (AppTweak + Ahrefs + Foreplay + Creatify)

---

## Ahrefs Unit Cost Model

**Plan**: $249/mo · 400,000 units/mo → **$0.0006225 / unit**
**Billing docs**: https://docs.ahrefs.com/en/api/docs/limits-consumption

### Formula
```
cost_per_call = max(50,  per_row_cost × num_rows)
per_row_cost  = sum of unit cost for each unique field in select + where + order_by
```
Cached responses consume **0 units**. Response headers: `x-api-units-cost-total-actual` = actual units charged.

### Field unit costs

| Cost | Fields |
|------|--------|
| **10 units** | `volume`, `difficulty`, `traffic`, `traffic_potential`, `intents`, `global_volume`, `parent_volume`, `volume_monthly`, `volume_monthly_history`, `sum_traffic`, `sum_paid_traffic`, `keyword_difficulty` (and their `_prev` / `_merged` variants) |
| **5 units** | `all_positions`, `all_positions_prev` |
| **1 unit** | everything else: `keyword`, `cpc`, `position`, `url`, `title`, `parent_topic`, `serp_features`, `first_seen`, etc. |

### Cost examples

| Scenario | Fields (per_row_cost) | Rows | Units | USD |
|----------|-----------------------|------|-------|-----|
| Expansion call (`select=keyword`) | 1 | 30 | **50** (min) | $0.03 |
| KW overview, 100 kw, typical select | 43 (7 fields incl. 4×10u) | 100 | **4,300** | $2.68 |
| KW overview, 500 kw, typical select | 43 | 500 | **21,500** | $13.40 |
| Organic keywords, 1,000 rows | 32 (volume+traffic+difficulty = 30u + 2×1u) | 1,000 | **32,000** | $19.94 |
| Domain rating only | 2 | 1 | **50** (min) | $0.03 |

**Key insight**: limiting `rows` and removing 10-unit fields from `select` is the single biggest cost lever.

### Free endpoints (0 units)
Management, Rank Tracker overview, Web Analytics, GSC Insights, Social Media, Public, Subscription Info.

---

## Ahrefs Skills

### ahrefs-keyword-discovery
Reference: https://docs.ahrefs.com/docs/api/reference/introduction

| Endpoint | Trigger | Calls/run | Units/call | USD/call |
|----------|---------|-----------|------------|----------|
| `/keywords-explorer/matching-terms` | Per seed keyword | N seeds (max 3) | 50 (min) — select=keyword only | $0.03 |
| `/keywords-explorer/related-terms` | Per seed keyword | N seeds (max 3) | 50 (min) — select=keyword only | $0.03 |
| `/keywords-explorer/search-suggestions` | Per seed keyword | N seeds (max 3) | 50 (min) — select=keyword only | $0.03 |
| `/keywords-explorer/overview` | Bulk enrichment (≤1000 kw/call) | 1–2 | 2,150–7,740 — 43u/row × 50-180 kw | $1.34–$4.82 |
| `/keywords-explorer/volume-history` | Top 5 keywords by volume | 5 | 50 (min) per call | $0.03 |

**Expansion calls** (matching/related/suggestions): `select=keyword` only → per_row_cost = 1 → each call = **50 units** (minimum). **3 seeds** (max) × 3 endpoints = 9 calls = **450 units** ($0.28).

**Overview enrichment**: typical `select=keyword,volume,difficulty,cpc,intents,traffic_potential,parent_topic` → per_row_cost = 43 (4 × 10u fields + 3 × 1u) → 50–180 unique kw after dedup → 2,150–7,740 units.

**Volume history** (top 5 kw): 5 × 50 units minimum = **250 units** ($0.16).

Typical total: **3,260–8,850 units/run** ($2.03–$5.51) · realistic max 22,910 units ($14.27 with 500 kw override)
Abort rule: any single call > 500 units → abort immediately.

---

### keyword-spy
| Endpoint | Trigger | Calls/run | Units/call | USD/call |
|----------|---------|-----------|------------|----------|
| `/site-explorer/organic-keywords` | Organic or both mode | 1 per domain | 1,020–3,400 — 34u/row × 30-100 rows | $0.64–$2.12 |
| `/site-explorer/paid-pages` | Paid or both mode | 1 per competitor | 640–3,200 — 32u/row × 20-100 rows | $0.40–$1.99 |
| `/keywords-explorer/overview` | Budget forecast (top 10 gap kw) | 1 | 1,290–4,300 — 43u/row × 30-100 kw | $0.80–$2.68 |
| `/serp-overview/serp-overview` | Head-to-head (top 20 paid gap kw) | ≤20 | 50 (min) per keyword | $0.03 each |
| `/site-explorer/top-pages` | Always | 1 per domain | ≥50 | ≥$0.03 |
| `/site-explorer/organic-competitors` | Always | 1 | ≥50 | ≥$0.03 |
| `/site-explorer/keywords-history` | Snapshot mode | 1 per domain | 1,020–3,400 — same select as organic-keywords | $0.64–$2.12 |
| `/site-explorer/metrics` | Always | 1 per domain | ≥50 | ≥$0.03 |
| `/batch-analysis` (POST) | Bulk domain metrics | 1 | ≥50 | ≥$0.03 |
| DataForSEO `domain_rank_overview` | **Optional** — only if Ahrefs shows 0 paid pages | 1 per zero-result domain | — | $0.0101/call |

Typical total (both mode, 1 competitor): **3,200–9,000 units/run** ($1.99–$5.60)

---

### backlink-profiler
| Endpoint | Trigger | Calls/run | Units/call | USD/call |
|----------|---------|-----------|------------|----------|
| `/site-explorer/metrics` | Always | 1 | ≥50 | ≥$0.03 |
| `/site-explorer/domain-rating` | Always | 1 | ≥50 | ≥$0.03 |
| `/site-explorer/backlinks-stats` | Always | 1 | ≥50 | ≥$0.03 |
| `/site-explorer/refdomains` | Always | 1 | ≥50 (10u fields, scales with rows) | ≥$0.03 |
| `/site-explorer/anchors` | Always | 1 | ≥50 | ≥$0.03 |
| `/site-explorer/refdomains-history` | Always | 1 | ≥50 (scales with rows) | ≥$0.03 |
| `/site-explorer/domain-rating-history` | Always | 1 | ≥50 | ≥$0.03 |
| `/site-explorer/broken-backlinks` | Always | 1 | ≥50 | ≥$0.03 |
| `/batch-analysis` (POST) | Cohort comparison (Step 6) | 1 | ≥50 | ≥$0.03 |

Typical total: **400–6,000 units/run** ($0.25–$3.74) — high end driven by refdomains/backlinks rows at 10u fields

---

### find-competitors
| Endpoint | Trigger | Calls/run | Units/call | USD/call |
|----------|---------|-----------|------------|----------|
| `/site-explorer/organic-competitors` | Always | 1 | ≥50 | ≥$0.03 |
| `/serp-overview/serp-overview` | Per seed keyword for SERP validation | 1–5 | 50 (min) per keyword | $0.03 each |
| `/site-explorer/top-pages` | Always | 1 | ≥50 | ≥$0.03 |
| `/site-explorer/metrics` | Per discovered competitor | 3–5 | ≥50 | ≥$0.03 |
| `/keywords-explorer/overview` | Per competitor keyword enrichment | 1 per competitor | 1,290–4,300 — 43u/row × 30-100 kw | $0.80–$2.68 |

Typical total: **400–1,500 units/run** ($0.25–$0.93)

---

### site-audit
| Endpoint | Trigger | Calls/run | Units/call | USD/call |
|----------|---------|-----------|------------|----------|
| `/site-audit/projects` | Always — resolve project_id | 1 | 0 | **Free** |
| `/site-audit/issues` | Always | 1 | 50 (fixed) | $0.03 |
| `/site-audit/page-explorer` | Always | 1 | 50 (fixed) | $0.03 |
| Firecrawl `/v1/scrape` | Tier B deep audit (top 3 pages) | 3–5 | — | $0.002/page |
| Google PageSpeed `/runPagespeed` | Tier B deep audit | 1 per page | — | **Free** |

Typical total: **100 Ahrefs units ($0.06) + 3–5 Firecrawl scrapes ($0.006–$0.01)**

---

### trend-analyzer
| Endpoint | Trigger | Calls/run | Units/call | USD/call |
|----------|---------|-----------|------------|----------|
| `/keywords-explorer/volume-history` | Per keyword | N keywords (max 5) | 50 (min) per call | $0.03 |
| `/keywords-explorer/volume-by-country` | Per keyword | N keywords | 50 (min) per call | $0.03 |
| `/keywords-explorer/overview` | Bulk current metrics | 1 | 2,150–4,300 — 43u/row × 50-100 kw | $1.34–$2.68 |
| `/serp-overview/serp-overview` | SERP snapshot per keyword | N keywords | 50 (min) per keyword | $0.03 each |

Typical total: **550–2,200 units/run** ($0.34–$1.37) — overview call dominates at 43 units/row × up to 50 rows

---

### serp-ads-monitor
| Endpoint | Trigger | Calls/run | Units/call | USD/call |
|----------|---------|-----------|------------|----------|
| `/serp-overview/serp-overview` | Per keyword | N keywords | 50 (min) per keyword | $0.03 each |
| `/keywords-explorer/overview` | Batch enrichment (≤100 kw/call) | 1 | 1,290–4,300 — 43u/row × 30-100 kw | $0.80–$2.68 |
| DataForSEO `domain_rank_overview` | **Optional** — only if Ahrefs shows 0 paid slots | 1 per zero-result keyword | — | $0.0101/call |

Typical total: **500–4,300 units/run** ($0.31–$2.68) — overview call at 43u/row × 100 rows = 4,300 units

---

### landing-page-spy
| Endpoint | Trigger | Calls/run | Units/call | USD/call |
|----------|---------|-----------|------------|----------|
| `/site-explorer/paid-pages` | Per competitor domain | 1 per competitor | 640–3,200 — 32u/row × 20-100 rows | $0.40–$1.99 |
| `/keywords-explorer/overview` | Batch keyword enrichment (≤100 kw/call) | 1–2 | 1,290–4,300 — 43u/row × 30-100 kw | $0.80–$2.68 |
| Firecrawl `/v1/scrape` | Tier A: unknown-type pages; Tier B: top 3 by traffic | 1–10 per competitor | — | $0.002/page |
| Google PageSpeed `/runPagespeed` | Tier B only | 1–3 per competitor | — | **Free** |
| DataForSEO `domain_rank_overview` | **Optional** — only if Ahrefs shows 0 paid pages | 1 per zero-result domain | — | $0.0101/call |

Typical total: **600–5,000 Ahrefs units ($0.37–$3.11) + 3–10 Firecrawl scrapes ($0.006–$0.02)** — high end driven by paid-pages (32u/row × 100 rows = 3,200 units) + kw overview (43u/row × 100 rows = 4,300 units)

---

### mention-tracker
| Endpoint | Trigger | Calls/run | Units/call | USD/call |
|----------|---------|-----------|------------|----------|
| `POST /management/brand-radar-reports` | Create report if not exists | 0–1 | 0 | **Free** |
| `GET /management/brand-radar-reports` | List existing reports | 1 | 0 | **Free** |
| `POST /management/brand-radar-prompts` | Configure prompts | 0–1 | 0 | **Free** |
| `GET /management/brand-radar-prompts` | List prompts | 1 | 0 | **Free** |
| `PATCH /management/brand-radar-reports` | Update prompts on report | 0–1 | 0 | **Free** |
| `/brand-radar/mentions-overview` | Always | 1 | ≥50 | ≥$0.03 |
| `/brand-radar/mentions-history` | Always | 1 | ≥50 | ≥$0.03 |
| `/brand-radar/sov-overview` | Always | 1 | ≥50 | ≥$0.03 |
| `/brand-radar/sov-history` | Always | 1 | ≥50 | ≥$0.03 |
| `/brand-radar/impressions-overview` | Always | 1 | ≥50 | ≥$0.03 |
| `/brand-radar/ai-responses` | Always | 1 per data_source (1–7) | ≥50 | ≥$0.03 |
| `/brand-radar/cited-pages` | Always | 1 | ≥50 | ≥$0.03 |
| `/brand-radar/cited-domains` | Always | 1 | ≥50 | ≥$0.03 |

Typical total: **200–600 units/run** ($0.12–$0.37) — management calls are free; paid calls are all minimum 50 units

---

### ads-media-plan

#### `--platform google`

| Sub-skill / Endpoint | Trigger | Calls/run | USD/call |
|----------------------|---------|-----------|----------|
| `/find-competitors` (sub-skill) | Always | 1 invocation | ~$0.59 · see find-competitors |
| `/keyword-spy --mode paid` (sub-skill) | Always (skipped if competitive_google_ads_analysis.md exists) | 1 invocation | ~$2.20 · see keyword-spy |
| `/serp-ads-monitor` (sub-skill) | Always | 1 invocation | ~$1.49 · see serp-ads-monitor |
| `/landing-page-spy` (sub-skill) | Always | 1 invocation | ~$1.74 · see landing-page-spy |
| `/site-explorer/paid-pages` | Direct — keyword annotation | 1 per competitor | $0.40–$1.99 |
| `/keywords-explorer/overview` | Direct — CPC enrichment | 1 | $0.80–$2.68 |

**End-to-end total (Google, standalone)**: ~$7.00–$9.50/run

#### `--platform meta`

| Sub-skill / Endpoint | Trigger | Calls/run | USD/call |
|----------------------|---------|-----------|----------|
| `/find-competitors` (sub-skill) | Always | 1 invocation | ~$0.59 · see find-competitors |
| `/ad-creative-spy` (sub-skill) | Per competitor | 1 per competitor | ~$0.30 total · see ad-creative-spy |
| `/landing-page-spy` (sub-skill) | Always | 1 invocation | ~$1.74 · see landing-page-spy |
| `/site-explorer/paid-pages` | Direct — paid page annotations | 1 per competitor | $0.40–$1.99 |

**End-to-end total (Meta)**: ~$2.80–$3.80/run

---

### geo-analysis

Orchestration skill — calls `/mention-tracker --channels ai` (2–3 invocations). No direct Ahrefs endpoint calls.

| Sub-skill | Trigger | Calls/run | USD |
|-----------|---------|-----------|-----|
| `/mention-tracker` (own domain, standard) | Step 1 baseline | 1 | ~$0.06 |
| `/mention-tracker` (own domain, deep) | Step 2 deep audit | 1 | ~$0.12 |
| `/mention-tracker` (per competitor, deep) | Step 3 (2–3 competitors) | 2–3 | ~$0.12 each |

Typical total: **~190–600 units/run** ($0.12–$0.37)

---

### competitive-analysis

Orchestration skill — cost = sum of sub-skills invoked per scope.

| Scope | Sub-skills | End-to-end estimate |
|-------|-----------|---------------------|
| `--scope ads` | find-competitors + ad-creative-spy + landing-page-spy + generate-ad-creative | ~$2.50–$3.30 |
| `--scope google-ads` | find-competitors + keyword-spy + serp-ads-monitor + landing-page-spy | ~$5.00–$7.00 |
| `--scope geo` | find-competitors + geo-analysis | ~$0.60–$1.00 |
| `--scope all` | all three scopes in parallel (sub-skills shared where possible) | ~$5.50–$8.00 |

**Note**: `--scope all` does **not** invoke `app-store-spy`. Step 2-pre (AppTweak) runs only for app clients and uses a subset of AppTweak endpoints directly (market detection + competitor metrics + reviews) — not the full health mode. Add ~$1.50 AppTweak for app clients.

---

## AppTweak Skills

### app-store-spy
Reference: `skills/app-store-spy/references/apptweak-api.md`
**Rate**: $199/mo · 250,000 credits → **$0.000796/credit**

| Endpoint | Trigger | Calls/run | Credits/call | USD/call |
|----------|---------|-----------|-------------|----------|
| `/usage/credits` | Pre-flight + post-run | 2 | 0 | **Free** |
| `/apps/category-rankings/current.json` | Step 1 market scan (24 markets) | 24 | ~11/app | ~$0.009 |
| `/apps/metadata.json` | Health mode Step 3 | 1 per app | ~11 | ~$0.009 |
| `/apps/metadata/changes.json` | Health mode Step 3 | 1 per app | ~41 (30d) | ~$0.033 |
| `/apps/category-rankings/current.json` | Health mode Step 3 (confirmed markets) | 1 per app × market | ~11 | ~$0.009 |
| `/apps/category-rankings/history.json` | Health mode Step 3 | 1 per app × market | ~40 (30d, 1 app) | ~$0.032 |
| `/apps/reviews/stats.json` | Health / reviews mode | 1 per app × market | ~39 (30d) · ~196 (30d, 5 apps) | ~$0.031–$0.156 |
| `/apps/reviews/top-displayed.json` | Health mode (×2 sorts) / reviews mode | 2 per app × market | ~101/call (limit=100) | ~$0.080 |
| `/apps/metrics/current.json` | Health mode Step 3 | 1 per app | ~516 (4 metrics) | **~$0.411** |
| `/apps/metrics/history.json` | Health mode Step 3 | 1 per app | ~516 (30d) | **~$0.411** |
| `/keywords/suggestions/app.json` | Health / keywords mode Step 4/6 | 1 per app | ~51 | ~$0.041 |
| `/keywords/metrics/current.json` | Health / keywords mode (batches of 5) | ceil(N/5) | ~251/call (5kw×6metrics) | ~$0.200 |
| `/apps/keywords-rankings/current.json` | Keywords mode Step 9 | 1 per app | 251–751 | $0.200–$0.598 |
| `/keywords/suggestions/category.json` | Keywords mode Step 9 | 0–1 | ~501 | ~$0.399 |
| `/charts/top-results/current.json` | Chart mode Step 13 | 1 per market | ~11 | ~$0.009 |
| `/charts/top-results/history.json` | Chart mode Step 13 (optional) | 0–1 per market | varies | varies |
| `/apps/reviews/search.json` | Reviews mode Step 12 | 1 per term × app × market | ~101 | ~$0.080 |
| `/apps/reviews/top-displayed.json` (sort=most_useful) | Reviews mode Step 12b (UGC hooks) | 1 per app × country | ~101 | ~$0.080 |
| `/apps/keywords/bids.json` | **`--probe-paid` only** | 1 per app | ~1,095 | **~$0.871** ⚠️ |
| `/keywords/apps/bids.json` (SOV) | **`--probe-paid` only** | 1 per 5 kw | ~121 (5kw/30d) | ~$0.096 |

**Estimated credits and USD per mode (1 app, 1 confirmed market):**

| Mode | Credits | USD | Notes |
|------|---------|-----|-------|
| health (default) | ~4,200 | **~$3.34** | metrics_current/history (~1,032) + keyword enrichment (~2,510 for 50 kw) + pre-flight scan (~264) |
| health + `--probe-paid` | +1,095/app | +**$0.87**/app | paid_keywords; SOV adds ~121 cr (+$0.10) |
| keywords | ~1,500–2,500 | **~$1.19–$1.99** | keyword_suggestions + keyword_metrics batches |
| chart | ~11–50 | **~$0.01–$0.04** | Cheapest mode |
| reviews | ~500–700 | **~$0.40–$0.56** | reviews_stats + reviews_top × 2 |
| reviews + UGC 12b | +101/app/country | +**$0.08**/app/country | Optional step |

**Monthly budget headroom (250,000 cr/mo):**
- health mode only: ~59 runs/month (1 app each)
- chart mode only: ~5,000 runs/month

---

## Foreplay Skills

**Rate**: $99/mo · 100,000 credits/mo → **$0.00099/credit**
Credit model: 1 credit per ad returned. Failed requests do NOT consume credits.

### ad-creative-spy

| Endpoint | Trigger | Calls/run | Credits/call | USD/call |
|----------|---------|-----------|-------------|----------|
| `GET /v2/brands/domain` | Pre-flight brand lookup | 1 per competitor | 0 | **Free** |
| `GET /v2/brands/{id}/ads` | Ad library pull | 1 per competitor | ~50 cr (limit=50 ads) | ~$0.050 |
| `POST /v2/search/ads` | Discovery search (additional terms) | 0–1 per competitor | ~50 cr (limit=50 ads) | ~$0.050 |

Typical: 3 competitors × 100 cr each = **~300 credits/run** (~$0.30)
High end: 5 competitors × 100 cr + 1 discovery search = **~600 credits/run** (~$0.59)

**Monthly headroom (100,000 cr/mo):** ~333 runs/month (3 competitors each)

---

## Firecrawl Skills

**Rate**: ~$0.002/page (Starter plan)

| Skill | Endpoint | Trigger | Calls/run | USD/call |
|-------|----------|---------|-----------|---------|
| site-audit | `/v1/scrape` | Tier B (top 3 pages) | 3–5 | $0.002 |
| landing-page-spy | `/v1/scrape` | Tier A (unknown pages) + Tier B (top 3) | 1–10/competitor | $0.002 |
| generate-ad-creative | `/v1/scrape` | Competitor page reference | 1–5 | $0.002 |

---

## Media / Creative Skills

### generate-ad-creative

| Provider | Endpoint | Trigger | Calls/run | USD/call |
|----------|----------|---------|-----------|---------|
| Firecrawl | `/v1/scrape` | Competitor reference scrape | 1–5 | $0.002/page |
| ScreenshotOne | `GET /take` | Style reference screenshot | 1–3 | $0.003/screenshot |
| Gemini | `generateContent` (gemini-3.1-flash-image-preview) | Hero + variations + resizes | 1 + N + M | **$0.067/image** (1024×1024 · $60/1M output tokens · ~1,120 tokens/img) |

---

### clip-video

| Provider | Endpoint | Trigger | Calls/run | USD/call |
|----------|----------|---------|-----------|---------|
| Deepgram | `POST /v1/listen` (Nova-2) | Audio transcription | 1 | $0.0059/min |
| yt-dlp | local binary | URL input only | 0–1 | Free |
| ffmpeg | local binary | Always | 1 (extract) + N×M (clips) | Free |

Typical: 5-min video = 5 min × $0.0059 = **~$0.030**

---

### video-creative-pipeline

**Creatify rate**: $0.198/credit (Starter: $99/mo ÷ 500 cr)

| Provider | Endpoint | Trigger | Calls/run | USD/call |
|----------|----------|---------|-----------|---------|
| Gemini | Files API upload | Always | 1 | Free |
| Gemini | `generateContent` (2.5 Pro) | Always | 1 | varies by tokens |
| fal.ai Seedance 2.0 | `POST fal-ai/bytedance/seedance-2/text-to-video` | Track A | 1 | $0.30/s std · $0.24/s fast |
| fal.ai Seedance 2.0 | status polling | Track A polling | N | Free |
| ElevenLabs | `POST /v1/text-to-speech/{voice_id}` | Track A (narration) | 1 | ~$0.30/1K chars (TBC) |
| Creatify | `POST /personal_photo_avatar/` | Track B only | 1 | $0.198/s Aurora · $0.099/s Fast |
| Creatify | `POST /ai_scripts/` | Track B script gen | 1 | $0.198/request (1 cr) |
| Creatify | `POST /text_to_speech/` | Track B TTS | 1 | $0.198/30s (1 cr/30s) |
| Creatify | `POST /lipsyncs/` | Track B only | 1 | $0.198/s Aurora · $0.099/s Fast |
| Creatify | `GET /lipsyncs/{task_id}/` | Track B polling | N | Free |
| Deepgram | `POST /v1/listen` | Track B (B-roll sync) | 1 | $0.0059/min |
| Ahrefs (via sub-skills) | see find-competitors, ad-creative-spy | Always | per sub-skill | see Ahrefs table |

**Track B cost for a 10s video** (Aurora): avatar 10s ($1.98) + script ($0.20) + TTS 10s ($0.07) + lipsync 10s ($1.98) = **~$4.23** (~21.4 cr)

---

## DataForSEO (Cross-validation only)

**Usage pattern**: DataForSEO is **not** a primary data source in any skill. It fires only as a cross-validation fallback when Ahrefs returns 0 paid activity for a competitor — to verify whether the zero is real or a coverage gap.

**Auth**: `Authorization: Basic base64(LOGIN:PASSWORD)` — requires `DATAFORSEO_LOGIN` + `DATAFORSEO_PASSWORD` env vars. If not set, cross-validation is silently skipped.

| Endpoint | Path | Trigger | Used by | Cost |
|----------|------|---------|---------|------|
| `domain_rank_overview` | `POST /v3/dataforseo_labs/google/domain_rank_overview/live` | Ahrefs returns 0 paid pages/keywords | landing-page-spy, serp-ads-monitor, competitive-analysis | $0.01/task + $0.0001/item → **$0.0101/call** (1 domain) |

**Pricing tier**: DataForSEO Labs "All Other Endpoints" — $0.01/task + $0.0001/item. `include_clickstream_data=true` doubles cost (not used).

**Typical cost**: 0–3 calls/run → **$0–$0.03** — most runs will not call DataForSEO at all.

**Output used**: `tasks[0].result[0].metrics.paid.keywords` + `metrics.paid.traffic` — cross-validated against Ahrefs. "No paid activity" conclusion only when **both sources confirm zero**.

---

## Free APIs (no cost)

| API | Used by | Notes |
|-----|---------|-------|
| Google PageSpeed `/runPagespeed` | site-audit, landing-page-spy | Free, rate-limited |
| Ahrefs `/site-audit/projects` | site-audit | Free endpoint |
| AppTweak `/usage/credits` | app-store-spy | Free endpoint |
| AppTweak management endpoints | mention-tracker | All `/management/brand-radar-*` are free |
| fal.ai / Creatify polling | video-creative-pipeline | Status-check GETs are free |
| Gemini Files API upload | video-creative-pipeline | Upload is free; generation is metered |

---

## Rate Reference

| Provider | Unit | Reference rate | Notes |
|----------|------|---------------|-------|
| Ahrefs | API unit | **$0.0006225** ($249/mo ÷ 400K units) | `max(50, per_row_cost × rows)`; 10-unit fields dominate cost |
| AppTweak | credit | **$0.000796** (Small: $199/mo ÷ 250K cr) | Call `/usage/credits` before/after to verify actual usage |
| Foreplay | credit | **$0.00099** ($99/mo ÷ 100K cr) | 1 cr per ad returned |
| Creatify | credit | **$0.198** (Starter: $99/mo ÷ 500 cr) | Aurora 1cr/s · Fast 0.5cr/s · Script 1cr/req · TTS 1cr/30s |
| Firecrawl | page scrape | **$0.002** | Starter plan |
| ScreenshotOne | screenshot | **$0.003** | Pay-as-you-go |
| Deepgram Nova-2 | minute of audio | **$0.0059/min** ($0.35/hr · PAYG) | Nova-2 pay-as-you-go |
| ElevenLabs | 1K characters | **~$0.30/1K chars** (TBC) | PAYG — exact rate to be confirmed |
| Gemini 3.1 Flash Image | image (1024×1024) | **$0.067/image** | $60/1M output tokens · ~1,120 tokens/img |
| fal.ai Seedance 2.0 | second of video (720p) | **$0.30/s std · $0.24/s fast** | 10s clip ≈ $3.00 std / $2.40 fast; image-to-video ×0.6 discount |
| DataForSEO Labs | task + items | **$0.01/task + $0.0001/item** | `domain_rank_overview/live`: $0.0101/call. Labs "All Other Endpoints" tier. |
| DataForSEO SERP Live | query | **$0.002/query** | Not currently used by any skill |
| Google PageSpeed | — | **Free** | No charge |
