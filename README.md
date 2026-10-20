# Scrape TikTok at Scale: Videos, Comments, Profiles and Trends

TikTok's API is a walled garden with research-only access. The most-used scraper on Apify (`clockworks/tiktok-scraper`) has **348k users** for a reason: marketers, trend analysts, and UGC agencies all need the data. Here's the pipeline — videos, comments, profiles, hashtag trends.

> 🤖 **Try it live** — chat with a Poe bot that generates the exact pipeline for your niche: [Social Lead Finder](https://poe.com/socialleadfinder) · [Lead List Generator](https://poe.com/leadlistgenerator) · [Job Postings Miner](https://poe.com/jobpostingsminer) · [Amazon Product Miner](https://poe.com/amazonproductminer) · [SERP Rank Tracker](https://poe.com/serpranktracker)

## The stack

- **[Apify](https://console.apify.com/sign-up?fpr=dx06p)** — the Clockworks TikTok suite, battle-tested by 300k+ users. Free monthly credits.
- **Python** — trend detection, comment mining, creator data.

## Step 1 — Token

[Create an Apify account](https://console.apify.com/sign-up?fpr=dx06p) → Settings → Integrations → `export APIFY_TOKEN=...`

## Step 2 — Hashtag / search scraping

```bash
curl -X POST "https://api.apify.com/v2/acts/clockworks~tiktok-scraper/run-sync-get-dataset-items?token=$APIFY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"hashtags": ["skincareroutine"], "resultsPerPage": 100, "shouldDownloadVideos": false}'
```

Fields: `text`, `authorMeta.name`, `diggCount`, `shareCount`, `playCount`, `createTime`, `videoUrl`, `webVideoUrl`.

## Step 3 — Spot what's going viral

```python
from apify_client import ApifyClient
import datetime

client = ApifyClient("APIFY_TOKEN")
run = client.actor("clockworks/tiktok-scraper").call(run_input={
    "hashtags": ["your_niche_hashtag"], "resultsPerPage": 200,
})

for v in client.dataset(run["defaultDatasetId"]).iterate_items():
    plays = v.get("playCount", 0)
    age_h = (datetime.datetime.now().timestamp() - v["createTime"]) / 3600
    velocity = plays / max(age_h, 1)
    if velocity > 10000:  # >10k plays/hour = going viral NOW
        print("VIRAL:", v["text"][:60], "|", plays, "plays in", int(age_h), "h")
```

## The plays

- **UGC agencies** — pull a creator's catalog (`clockworks/tiktok-profile-scraper`), sort by views → their best hooks.
- **Trend jacking** — daily hashtag runs → catch trends at <48h old, before saturation.
- **Comments = voice of customer** — `clockworks/tiktok-comments-scraper` (48k users) on viral videos in your niche → objections, wants, language.
- **Lead gen** — bios with emails/links → contact-info enrichment.

## Useful actors

| Actor | Job | Users |
|---|---|---|
| `clockworks/tiktok-scraper` | Videos by hashtag/search/user | 348k |
| `clockworks/free-tiktok-scraper` | Free-tier extractor | 59.7k |
| `clockworks/tiktok-profile-scraper` | Creator catalogs | 56.8k |
| `clockworks/tiktok-comments-scraper` | Comment mining | 48.8k |

## More resources

- [apify-skills](https://github.com/g4dr/apify-skills) — `apify-social-scraper` skill runs TikTok pipelines from a prompt
- [awesome-web-scraping](https://github.com/g4dr/awesome-web-scraping) — curated tools
- Free credits: [Apify signup](https://console.apify.com/sign-up?fpr=dx06p)
