# API Cost Reference

Third-party API calls across all brand-template skills.
Maintained as the single source of truth for cost estimation and budget planning.

**Total fixed subscriptions: $646/mo** (AppTweak $199 + Ahrefs $249 + Foreplay $99 + Creatify $99)

---

## API Billing Policies

### Ahrefs

**Plan**: Standard · $249/mo · 400,000 units/mo → **$0.0006225 / unit**
**Billing docs**: https://docs.ahrefs.com/en/api/docs/limits-consumption

**Billing model**: units consumed per call = `max(50, per_row_cost × num_rows)`
- `per_row_cost` = sum of unit weights for each field in `select` + `where` + `order_by`
- Cached responses consume **0 units**
- Actual units charged: read `x-api-units-cost-total-actual` response header

**Field weights:**

| Weight | Fields |
|--------|--------|
| **10 units** | `volume`, `difficulty`, `traffic`, `traffic_potential`, `intents`, `global_volume`, `parent_volume`, `volume_monthly`, `volume_monthly_history`, `sum_traffic`, `sum_paid_traffic`, `keyword_difficulty` (and `_prev` / `_merged` variants) |
| **5 units** | `all_positions`, `all_positions_prev` |
| **1 unit** | `keyword`, `cpc`, `position`, `url`, `title`, `parent_topic`, `serp_features`, `first_seen`, and all other fields |

**Key insight**: removing 10-unit fields from `select` and capping `rows` are the two biggest cost levers.

#### Endpoint Cost Directory

All Ahrefs endpoints used across skills, with actual per-call cost based on the `select` fields we use.

**Keywords Explorer**

| Endpoint | Our select fields | per_row_cost | Typical rows | Units/call | USD/call |
|----------|-------------------|-------------|--------------|------------|----------|
| `/keywords-explorer/matching-terms` | `keyword` only (expansion) | 1 | 30 | **50** (min) | $0.03 |
| `/keywords-explorer/related-terms` | `keyword` only (expansion) | 1 | 30 | **50** (min) | $0.03 |
| `/keywords-explorer/search-suggestions` | `keyword` only (expansion) | 1 | 30 | **50** (min) | $0.03 |
| `/keywords-explorer/overview` | `keyword,volume,difficulty,cpc,intents,traffic_potential,parent_topic` | 43 (4×10u + 3×1u) | 50–180 | 2,150–7,740 | $1.34–$4.82 |
| `/keywords-explorer/volume-history` | `keyword,volume_monthly_history` | 11 | 1 per kw | **50** (min) | $0.03 |
| `/keywords-explorer/volume-by-country` | `keyword,volume` | 11 | 1 per kw | **50** (min) | $0.03 |

**Site Explorer**

| Endpoint | Our select fields | per_row_cost | Typical rows | Units/call | USD/call |
|----------|-------------------|-------------|--------------|------------|----------|
| `/site-explorer/organic-keywords` | `keyword,volume,cpc,difficulty,position,url,intents` | 34 (3×10u + 4×1u) | 30–100 | 1,020–3,400 | $0.64–$2.12 |
| `/site-explorer/paid-pages` | `url,traffic,cpc,keywords,top_keyword` | 32 (2×10u + 3×1u) | 20–100 | 640–3,200 | $0.40–$1.99 |
| `/site-explorer/keywords-history` | same as organic-keywords | 34 | 30–100 | 1,020–3,400 | $0.64–$2.12 |
| `/site-explorer/top-pages` | `url,traffic,value,keywords` | ~22 (2×10u + 2×1u) | 10–50 | 220–1,100 | $0.14–$0.69 |
| `/site-explorer/organic-competitors` | `competitor,common_keywords,competition_level` | ~3 | 10–20 | **50** (min) | $0.03 |
| `/site-explorer/metrics` | `domain_rating,ahrefs_rank,org_keywords,org_traffic` | ~22 | 1 | **50** (min) | $0.03 |
| `/site-explorer/domain-rating` | `domain_rating,ahrefs_rank` | 2 | 1 | **50** (min) | $0.03 |
| `/site-explorer/backlinks-stats` | `live,all_time,new_lost_links` | ~3 | 1 | **50** (min) | $0.03 |
| `/site-explorer/refdomains` | `domain,domain_rating,linked_domains,dofollow` | ~4 | 50–200 | **50**–800 | $0.03–$0.50 |
| `/site-explorer/refdomains-history` | `date,refdomains` | 2 | 30–365 | **60**–730 | $0.04–$0.45 |
| `/site-explorer/anchors` | `anchor,backlinks,dofollow,referring_domains` | ~4 | 20–100 | **80**–400 | $0.05–$0.25 |
| `/site-explorer/domain-rating-history` | `date,domain_rating` | 2 | 30–365 | **60**–730 | $0.04–$0.45 |
| `/site-explorer/broken-backlinks` | `url_from,url_to,ahrefs_rank` | ~3 | 10–50 | **50** (min) | $0.03 |
| `/batch-analysis` (POST) | `domain_rating,org_keywords,org_traffic` | ~22 | 1 per domain | **50** (min) | $0.03 |

**SERP Overview**

| Endpoint | Our select fields | per_row_cost | Typical rows | Units/call | USD/call |
|----------|-------------------|-------------|--------------|------------|----------|
| `/serp-overview/serp-overview` | `position,url,domain,traffic` | ~12 (1×10u + 2×1u) | 10 | **50** (min) per keyword | $0.03 |

**Site Audit**

| Endpoint | Notes | Units/call | USD/call |
|----------|-------|------------|----------|
| `/site-audit/projects` | Resolve project_id | 0 | **Free** |
| `/site-audit/issues` | Fixed cost | **50** | $0.03 |
| `/site-audit/page-explorer` | Fixed cost | **50** | $0.03 |

**Brand Radar**

| Endpoint | Notes | Units/call | USD/call |
|----------|-------|------------|----------|
| All `/management/brand-radar-*` | Create / list / update reports and prompts | 0 | **Free** |
| `/brand-radar/mentions-overview` | Per domain | **50** (min) | $0.03 |
| `/brand-radar/mentions-history` | Per domain | **50** (min) | $0.03 |
| `/brand-radar/sov-overview` | Per domain | **50** (min) | $0.03 |
| `/brand-radar/sov-history` | Per domain | **50** (min) | $0.03 |
| `/brand-radar/impressions-overview` | Per domain | **50** (min) | $0.03 |
| `/brand-radar/ai-responses` | Per `data_source` (chatgpt / perplexity / google_ai / gemini) | **50** (min) each | $0.03 each |
| `/brand-radar/cited-pages` | Per domain | **50** (min) | $0.03 |
| `/brand-radar/cited-domains` | Per domain | **50** (min) | $0.03 |

---

### AppTweak

**Plan**: Small · $199/mo · 250,000 credits/mo → **$0.000796 / credit**
**Billing model**: credits charged per response row or per result set. Failed requests do NOT consume credits.
**Docs**: https://developers.apptweak.com/reference/overview

| Endpoint | Credits/call | USD/call | Notes |
|----------|-------------|----------|-------|
| `/usage/credits` | 0 | **Free** | Pre-flight + post-run check |
| `/apps/category-rankings/current.json` | ~11/app | ~$0.009 | Market scan; 24 calls for full scan |
| `/apps/metadata.json` | ~11 | ~$0.009 | App metadata |
| `/apps/metadata/changes.json` | ~41 (30d) | ~$0.033 | Metadata change history |
| `/apps/category-rankings/history.json` | ~40 (30d, 1 app) | ~$0.032 | Ranking history per market |
| `/apps/reviews/stats.json` | ~39–196 (30d, 1–5 apps) | ~$0.031–$0.156 | Review stats |
| `/apps/reviews/top-displayed.json` | ~101/call (limit=100) | ~$0.080 | Top reviews; called ×2 sorts |
| `/apps/metrics/current.json` | ~516 (4 metrics, 1 app) | **~$0.411** | Downloads/revenue — most expensive routine call |
| `/apps/metrics/history.json` | ~516 (30d, 1 app) | **~$0.411** | 30-day metrics history |
| `/keywords/suggestions/app.json` | ~51 | ~$0.041 | App keyword suggestions |
| `/keywords/metrics/current.json` | ~251/call (5kw × 6metrics) | ~$0.200 | Batches of 5 keywords |
| `/apps/keywords-rankings/current.json` | 251–751 | $0.200–$0.598 | Keyword rankings per app |
| `/keywords/suggestions/category.json` | ~501 | ~$0.399 | Category keyword suggestions |
| `/charts/top-results/current.json` | ~11/market | ~$0.009 | Chart rankings |
| `/charts/top-results/history.json` | varies | varies | Optional; chart history |
| `/apps/reviews/search.json` | ~101/call | ~$0.080 | Reviews search per term |
| `/apps/reviews/top-displayed.json` (sort=most_useful) | ~101/call | ~$0.080 | UGC hooks (optional step 12b) |
| `/apps/keywords/bids.json` | ~1,095/app | **~$0.871** ⚠️ | `--probe-paid` only |
| `/keywords/apps/bids.json` (SOV) | ~121 (5kw/30d) | ~$0.096 | `--probe-paid` only |

---

### Foreplay

**Plan**: Pro · $99/mo · 100,000 credits/mo → **$0.00099 / credit**
**Billing model**: 1 credit per ad returned. Failed requests do NOT consume credits.
**Docs**: https://foreplay.co/api

| Endpoint | Credits/call | USD/call | Notes |
|----------|-------------|----------|-------|
| `GET /v2/brands/domain` | 0 | **Free** | Brand lookup by domain |
| `GET /v2/brands/{id}/ads` | ~50 cr (limit=50 ads) | ~$0.050 | Ad library pull per competitor |
| `POST /v2/search/ads` | ~50 cr (limit=50 ads) | ~$0.050 | Discovery search |

---

### Creatify

**Plan**: Starter · $99/mo · 500 credits/mo → **$0.198 / credit**
**Billing model**: credits charged per second of generated video / per request.
**Docs**: https://docs.creatify.ai/api-reference/introduction

| Endpoint | Credits/call | USD/call | Notes |
|----------|-------------|----------|-------|
| `POST /personal_photo_avatar/` | 1 cr/s (Aurora) · 0.5 cr/s (Fast) | $0.198/s · $0.099/s | Avatar video generation |
| `POST /ai_scripts/` | 1 cr/request | $0.198 | Script generation |
| `POST /text_to_speech/` | 1 cr/30s | $0.198/30s | TTS audio |
| `POST /lipsyncs/` | 1 cr/s (Aurora) · 0.5 cr/s (Fast) | $0.198/s · $0.099/s | Lip-sync generation |
| `GET /lipsyncs/{task_id}/` | 0 | **Free** | Status polling |

A 10-second Track B video (Aurora): avatar 10s + lipsync 10s + script + TTS ≈ **~21 cr / ~$4.20**

---

### DataForSEO

**Plan**: Pay-as-you-go
**Billing model**: $0.01/task + $0.0001/item returned (Labs endpoints). `include_clickstream_data=true` doubles cost — not used.
**Auth**: `Authorization: Basic base64(LOGIN:PASSWORD)` · env vars: `DATAFORSEO_LOGIN` + `DATAFORSEO_PASSWORD`
**Docs**: https://docs.dataforseo.com/v3/

| Endpoint | Path | Credits/call | USD/call | Notes |
|----------|------|-------------|----------|-------|
| `domain_rank_overview` | `POST /v3/dataforseo_labs/google/domain_rank_overview/live` | $0.01/task + $0.0001/item | **$0.0101/call** | 1 domain per call; Labs "All Other Endpoints" tier |

**Usage pattern**: cross-validation fallback only — fires when Ahrefs returns 0 paid activity for a competitor. If env vars not set, silently skipped.

---

### fal.ai

**Plan**: Pay-as-you-go
**Docs**: https://fal.ai/docs

| Model / Endpoint | Rate | Notes |
|-----------------|------|-------|
| Seedance 2.0 text-to-video (720p std) | **$0.30/s** | 10s clip ≈ $3.00 |
| Seedance 2.0 text-to-video (720p fast) | **$0.24/s** | 10s clip ≈ $2.40 |
| Seedance 2.0 image-to-video | ×0.6 discount | 10s ≈ $1.80 std / $1.44 fast |
| Status polling (GET) | Free | — |

---

### Deepgram

**Plan**: Pay-as-you-go · Nova-2 model
**Docs**: https://developers.deepgram.com/reference/deepgram-api-overview

| Endpoint | Rate | Notes |
|----------|------|-------|
| `POST /v1/listen` (Nova-2) | **$0.0059/min** ($0.35/hr) | Pre-recorded audio transcription |

Typical: 5-min video = **~$0.030**

---

### ElevenLabs

**Plan**: Pay-as-you-go
**Docs**: https://elevenlabs.io/docs/api-reference/introduction

| Endpoint | Rate | Notes |
|----------|------|-------|
| `POST /v1/text-to-speech/{voice_id}` | **~$0.30/1K chars** (TBC) | Exact PAYG rate to be confirmed |

---

### Firecrawl

**Plan**: Starter · usage-based → **~$0.002 / page scrape**
**Docs**: https://docs.firecrawl.dev/api-reference/v2-introduction

| Endpoint | Rate | Notes |
|----------|------|-------|
| `POST /v1/scrape` | **$0.002/page** | Used for competitor page scraping |

---

### ScreenshotOne

**Plan**: Pay-as-you-go → **~$0.003 / screenshot**
**Docs**: https://screenshotone.com/docs/getting-started/

| Endpoint | Rate | Notes |
|----------|------|-------|
| `GET /take` | **$0.003/screenshot** | Style reference screenshots |

---

### Google PageSpeed

**Free** (rate-limited). No account or payment required.
**Docs**: https://developers.google.com/speed/docs/insights/v5/reference/pagespeedapi/runpagespeed

---

## Skill Cost Breakdowns

### ahrefs-keyword-discovery

| Endpoint | Calls/run | USD/call | Subtotal |
|----------|-----------|----------|---------|
| `/keywords-explorer/matching-terms` | 3 (1/seed, max 3 seeds) | $0.03 | $0.09 |
| `/keywords-explorer/related-terms` | 3 | $0.03 | $0.09 |
| `/keywords-explorer/search-suggestions` | 3 | $0.03 | $0.09 |
| `/keywords-explorer/overview` | 1–2 | $1.34–$4.82 | $1.34–$9.64 |
| `/keywords-explorer/volume-history` | 5 (top 5 kw) | $0.03 | $0.15 |

**Typical total**: $2.03–$5.51/run · realistic max $14.27 (500 kw override)
Abort rule: any single call > 500 units → abort immediately.

---

### keyword-spy

| Endpoint | Calls/run | USD/call | Subtotal |
|----------|-----------|----------|---------|
| `/site-explorer/organic-keywords` | 1–2 (own + competitor) | $0.64–$2.12 | $0.64–$4.24 |
| `/site-explorer/paid-pages` | 1/competitor (paid/both mode) | $0.40–$1.99 | $0.40–$1.99 |
| `/site-explorer/keywords-history` | 1/domain (snapshot mode) | $0.64–$2.12 | $0.64–$2.12 |
| `/site-explorer/top-pages` | 1/domain | $0.14–$0.69 | $0.14–$0.69 |
| `/site-explorer/organic-competitors` | 1 | $0.03 | $0.03 |
| `/site-explorer/metrics` | 1/domain | $0.03 | $0.03 |
| `/keywords-explorer/overview` | 1 (budget forecast) | $0.80–$2.68 | $0.80–$2.68 |
| `/serp-overview/serp-overview` | ≤20 (paid gap kw) | $0.03 each | ≤$0.60 |
| `/batch-analysis` | 1 | $0.03 | $0.03 |
| DataForSEO `domain_rank_overview` | 0–1 (optional fallback) | $0.0101 | $0–$0.01 |

**Typical total**: $1.99–$5.60/run (both mode, 1 competitor)

---

### backlink-profiler

| Endpoint | Calls/run | USD/call | Subtotal |
|----------|-----------|----------|---------|
| `/site-explorer/metrics` | 1 | $0.03 | $0.03 |
| `/site-explorer/domain-rating` | 1 | $0.03 | $0.03 |
| `/site-explorer/backlinks-stats` | 1 | $0.03 | $0.03 |
| `/site-explorer/refdomains` | 1 | $0.03–$0.50 | $0.03–$0.50 |
| `/site-explorer/anchors` | 1 | $0.05–$0.25 | $0.05–$0.25 |
| `/site-explorer/refdomains-history` | 1 | $0.04–$0.45 | $0.04–$0.45 |
| `/site-explorer/domain-rating-history` | 1 | $0.04–$0.45 | $0.04–$0.45 |
| `/site-explorer/broken-backlinks` | 1 | $0.03 | $0.03 |
| `/batch-analysis` | 1 (cohort comparison) | $0.03 | $0.03 |

**Typical total**: $0.25–$3.74/run — high end driven by refdomains/backlinks at large row counts

---

### find-competitors

| Endpoint | Calls/run | USD/call | Subtotal |
|----------|-----------|----------|---------|
| `/site-explorer/organic-competitors` | 1 | $0.03 | $0.03 |
| `/serp-overview/serp-overview` | 1–5 (SERP validation) | $0.03 each | $0.03–$0.15 |
| `/site-explorer/top-pages` | 1 | $0.14–$0.69 | $0.14–$0.69 |
| `/site-explorer/metrics` | 3–5 (per competitor) | $0.03 each | $0.09–$0.15 |
| `/keywords-explorer/overview` | 1/competitor | $0.80–$2.68 | $0.80–$2.68 |

**Typical total**: $0.25–$0.93/run

---

### site-audit

| Endpoint | Calls/run | USD/call | Subtotal |
|----------|-----------|----------|---------|
| `/site-audit/projects` | 1 | Free | — |
| `/site-audit/issues` | 1 | $0.03 | $0.03 |
| `/site-audit/page-explorer` | 1 | $0.03 | $0.03 |
| Firecrawl `/v1/scrape` | 3–5 (Tier B only) | $0.002 | $0.006–$0.01 |
| Google PageSpeed `/runPagespeed` | 1/page (Tier B only) | Free | — |

**Typical total**: $0.06 Ahrefs + $0.006–$0.01 Firecrawl

---

### trend-analyzer

| Endpoint | Calls/run | USD/call | Subtotal |
|----------|-----------|----------|---------|
| `/keywords-explorer/volume-history` | N kw (max 5) | $0.03 each | $0.03–$0.15 |
| `/keywords-explorer/volume-by-country` | N kw | $0.03 each | $0.03–$0.15 |
| `/keywords-explorer/overview` | 1 | $1.34–$2.68 | $1.34–$2.68 |
| `/serp-overview/serp-overview` | N kw | $0.03 each | $0.03–$0.15 |

**Typical total**: $0.34–$1.37/run

---

### serp-ads-monitor

| Endpoint | Calls/run | USD/call | Subtotal |
|----------|-----------|----------|---------|
| `/serp-overview/serp-overview` | N keywords | $0.03 each | $0.03–$0.60 |
| `/keywords-explorer/overview` | 1 (batch enrichment) | $0.80–$2.68 | $0.80–$2.68 |
| DataForSEO `domain_rank_overview` | 0–1/zero-result kw (optional) | $0.0101 | $0–$0.03 |

**Typical total**: $0.31–$2.68/run

---

### landing-page-spy

| Endpoint | Calls/run | USD/call | Subtotal |
|----------|-----------|----------|---------|
| `/site-explorer/paid-pages` | 1/competitor | $0.40–$1.99 | $0.40–$1.99 |
| `/keywords-explorer/overview` | 1–2 | $0.80–$2.68 | $0.80–$5.36 |
| Firecrawl `/v1/scrape` | 1–10/competitor | $0.002 | $0.002–$0.02 |
| Google PageSpeed `/runPagespeed` | 1–3/competitor (Tier B) | Free | — |
| DataForSEO `domain_rank_overview` | 0–1/zero-result domain (optional) | $0.0101 | $0–$0.01 |

**Typical total**: $0.37–$3.11/run (Ahrefs) + $0.006–$0.02 (Firecrawl)

---

### mention-tracker

| Endpoint | Calls/run | USD/call | Subtotal |
|----------|-----------|----------|---------|
| All `/management/brand-radar-*` | 3–5 | Free | — |
| `/brand-radar/mentions-overview` | 1 | $0.03 | $0.03 |
| `/brand-radar/mentions-history` | 1 | $0.03 | $0.03 |
| `/brand-radar/sov-overview` | 1 | $0.03 | $0.03 |
| `/brand-radar/sov-history` | 1 | $0.03 | $0.03 |
| `/brand-radar/impressions-overview` | 1 | $0.03 | $0.03 |
| `/brand-radar/ai-responses` | 1–7 (per data_source) | $0.03 each | $0.03–$0.21 |
| `/brand-radar/cited-pages` | 1 | $0.03 | $0.03 |
| `/brand-radar/cited-domains` | 1 | $0.03 | $0.03 |

**Typical total**: $0.12–$0.37/run

---

### ad-creative-spy

| Endpoint | Calls/run | USD/call | Subtotal |
|----------|-----------|----------|---------|
| Foreplay `GET /v2/brands/domain` | 1/competitor | Free | — |
| Foreplay `GET /v2/brands/{id}/ads` | 1/competitor | ~$0.050 | ~$0.15 (3 competitors) |
| Foreplay `POST /v2/search/ads` | 0–1/competitor | ~$0.050 | ~$0–$0.25 |

**Typical total**: ~$0.30/run (3 competitors) · high end ~$0.59 (5 competitors + discovery)

---

### generate-ad-creative

| Endpoint | Calls/run | USD/call | Subtotal |
|----------|-----------|----------|---------|
| Firecrawl `/v1/scrape` | 1–5 | $0.002 | $0.002–$0.01 |
| ScreenshotOne `GET /take` | 1–3 | $0.003 | $0.003–$0.009 |
| Gemini `generateContent` (Flash Image) | 1 + variations + resizes | $0.067/image | $0.07–$0.40+ |

---

### clip-video

| Endpoint | Calls/run | USD/call | Subtotal |
|----------|-----------|----------|---------|
| Deepgram `POST /v1/listen` (Nova-2) | 1 | $0.0059/min | varies |
| yt-dlp (local) | 0–1 | Free | — |
| ffmpeg (local) | 1 + N clips | Free | — |

Typical: 5-min video = **~$0.030**

---

### app-store-spy

**Reference**: `skills/app-store-spy/references/apptweak-api.md`

| Mode | Key endpoints called | Credits | USD |
|------|---------------------|---------|-----|
| health (default) | metrics_current/history + keyword enrichment + pre-flight scan | ~4,200 | **~$3.34** |
| health + `--probe-paid` | + bids endpoints | +1,095/app | +**$0.87**/app |
| keywords | keyword_suggestions + keyword_metrics batches | ~1,500–2,500 | **~$1.19–$1.99** |
| chart | category-rankings/current | ~11–50 | **~$0.01–$0.04** |
| reviews | reviews_stats + reviews_top × 2 | ~500–700 | **~$0.40–$0.56** |
| reviews + UGC 12b | + reviews/top-displayed (sort=most_useful) | +101/app/country | +**$0.08**/app/country |

**Monthly headroom** (250,000 cr/mo): ~59 health runs · ~5,000 chart runs

---

### video-creative-pipeline

**Track A** (fal.ai Seedance + ElevenLabs narration)

| Endpoint | Calls/run | USD/call | Subtotal |
|----------|-----------|----------|---------|
| Gemini Files API upload | 1 | Free | — |
| Gemini `generateContent` (2.5 Pro) | 1 | varies by tokens | ~$0.01–$0.10 |
| fal.ai Seedance 2.0 text-to-video | 1 | $0.30/s std · $0.24/s fast | $3.00–$3.60 (10s) |
| ElevenLabs `POST /v1/text-to-speech` | 1 | ~$0.30/1K chars | varies (TBC) |

**Track B** (Creatify avatar + lipsync)

| Endpoint | Calls/run | USD/call | Subtotal |
|----------|-----------|----------|---------|
| Creatify `POST /ai_scripts/` | 1 | $0.198 | $0.198 |
| Creatify `POST /text_to_speech/` | 1 | $0.198/30s | ~$0.07 (10s) |
| Creatify `POST /personal_photo_avatar/` | 1 | $0.198/s Aurora | $1.98 (10s) |
| Creatify `POST /lipsyncs/` | 1 | $0.198/s Aurora | $1.98 (10s) |
| Deepgram `POST /v1/listen` | 1 | $0.0059/min | ~$0.03 |

**Track B total (10s, Aurora)**: **~$4.20/run**

---

### ads-media-plan

**`--platform google`** (orchestration — calls 4 sub-skills + 2 direct endpoints)

| Sub-skill / Endpoint | Calls/run | USD |
|----------------------|-----------|-----|
| `/find-competitors` | 1 | ~$0.59 |
| `/keyword-spy --mode paid` | 1 (skipped if cached) | ~$2.20 |
| `/serp-ads-monitor` | 1 | ~$1.49 |
| `/landing-page-spy` | 1 | ~$1.74 |
| `/site-explorer/paid-pages` (direct) | 1/competitor | $0.40–$1.99 |
| `/keywords-explorer/overview` (direct) | 1 | $0.80–$2.68 |

**End-to-end total**: ~$7.00–$9.50/run

**`--platform meta`** (orchestration — calls 3 sub-skills + 1 direct endpoint)

| Sub-skill / Endpoint | Calls/run | USD |
|----------------------|-----------|-----|
| `/find-competitors` | 1 | ~$0.59 |
| `/ad-creative-spy` | 1 | ~$0.30 |
| `/landing-page-spy` | 1 | ~$1.74 |
| `/site-explorer/paid-pages` (direct) | 1/competitor | $0.40–$1.99 |

**End-to-end total**: ~$2.80–$3.80/run

---

### geo-analysis

Orchestration skill — calls `/mention-tracker` only, no direct API calls.

| Sub-skill | Calls/run | USD |
|-----------|-----------|-----|
| `/mention-tracker` (own domain, standard) — Step 1 | 1 | ~$0.06 |
| `/mention-tracker` (own domain, deep) — Step 2 | 1 | ~$0.12 |
| `/mention-tracker` (per competitor, deep) — Step 3 | 2–3 | ~$0.12 each |

**Typical total**: $0.12–$0.37/run

---

### competitive-analysis

Orchestration skill — cost = sum of scoped sub-skills.

| Scope | Sub-skills invoked | End-to-end estimate |
|-------|-------------------|---------------------|
| `--scope ads` | find-competitors + ad-creative-spy + landing-page-spy + generate-ad-creative | ~$2.50–$3.30 |
| `--scope google-ads` | find-competitors + keyword-spy + serp-ads-monitor + landing-page-spy | ~$5.00–$7.00 |
| `--scope geo` | find-competitors + geo-analysis | ~$0.60–$1.00 |
| `--scope all` | all three scopes, sub-skills shared where possible | ~$5.50–$8.00 |

**Note**: `--scope all` does **not** invoke `app-store-spy`. For app clients only, Step 2-pre calls a subset of AppTweak endpoints directly (market detection + competitor metrics + reviews) — add ~$1.50 AppTweak.
