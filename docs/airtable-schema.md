# TFC Dashboard — Airtable Schema Design

## Overview

This schema stores cross-platform content analytics for Christie Mealo's personal brand dashboard. Data flows in from Ghost API, LinkedIn (scraped), Substack (scraped), and Medium (scraped) via daily ingestion jobs.

## Design Principles

- Single table where possible (Muji's guidance)
- Flat structure for easy querying and charting
- One row per metric per platform per day
- Normalize only if foreign key relationships become necessary

## Primary Table: `analytics_daily`

| Field | Type | Options | Required | Description |
|-------|------|---------|----------|-------------|
| id | Auto Number | — | auto | Primary key |
| date | Date | — | yes | Day the metric was captured |
| platform | Single Select | `linkedin`, `ghost`, `substack`, `medium` | yes | Source platform |
| metric_type | Single Select | `impressions`, `engagement_rate`, `followers`, `subscribers`, `open_rate`, `click_rate`, `views`, `posts_count` | yes | What's being measured |
| value | Number (decimal) | — | yes | The metric value |
| post_title | Single Line Text | — | no | For per-post metrics |
| post_url | URL | — | no | Link to the specific post |
| notes | Long Text | — | no | Free-form context |

## Example Rows

| date | platform | metric_type | value | post_title | post_url |
|------|----------|-------------|-------|------------|----------|
| 2026-06-01 | linkedin | impressions | 4521 | | |
| 2026-06-01 | linkedin | engagement_rate | 0.042 | | |
| 2026-06-01 | linkedin | followers | 12847 | | |
| 2026-06-01 | ghost | open_rate | 0.38 | "AI in Healthcare Q2" | https://... |
| 2026-06-01 | ghost | subscribers | 3241 | | |
| 2026-06-01 | substack | views | 892 | | |

## Querying Patterns

**Dashboard Performance tab:**

- Filter by date range + group by platform → cross-channel comparison
- Filter by platform + sort by date → single-platform trend line
- Filter by metric_type = 'engagement_rate' → engagement comparison chart

**Top posts:**

- Filter where post_title is not empty → per-post metrics
- Sort by value descending where metric_type = 'views' → top posts

**Growth tracking:**

- Filter metric_type = 'followers' or 'subscribers' → follower/subscriber growth over time

## Future Normalization (if needed)

If we need richer post-level data (multiple metrics per post, tags, content body), split into:

- `platforms` — id, name, api_type
- `daily_metrics` — id, date, platform_id, metric_type, value
- `posts` — id, platform_id, title, url, published_date, content_snippet
- `post_metrics` — id, post_id, metric_type, value, date

But start with the single table and only split if the data demands it.

## Airtable Base Setup

- Base name: `TFC Dashboard Analytics`
- Table: `analytics_daily`
- Views to create:
  - "All Data" (grid, default)
  - "LinkedIn Only" (filtered)
  - "Ghost Only" (filtered)
  - "This Week" (filtered by date)
  - "Top Posts" (filtered where post_title not empty, sorted by value desc)

## Integration

- Airtable API key in `.env.local` as `AIRTABLE_API_KEY`
- Base ID in `.env.local` as `AIRTABLE_BASE_ID`
- Use `airtable` npm package or direct REST API via TanStack Query
