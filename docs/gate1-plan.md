# TFC Dashboard — Technical Plan & Architecture

**Danny Gardner & Kushagra Behl | Philly AI Lab Summer 2026**
**Last updated: May 26, 2026**

---

## Executive Summary

The TFC Dashboard is a two-tab personal brand dashboard built for Christie Mealo of Trifecta Enterprises. The **Performance** tab surfaces cross-platform analytics (LinkedIn, Ghost, Substack, Medium) pulled from an Airtable base. The **Ideas** tab delivers a curated article feed powered by a vector database, with AI features layered on top in later milestones. The stack is Next.js 15 + Tailwind + shadcn/ui, deployed on Vercel.

---

## Architecture

### Data Sources

Four content platforms feed the system:

1. **LinkedIn** — professional posts, impressions, engagement
2. **Ghost** — blog posts, subscriber metrics, email open/click rates
3. **Substack** — newsletter metrics, subscriber counts
4. **Medium** — article views, reads, engagement

### "The Brain" — Dual Data Store

All collected data flows into two complementary stores:

- **Airtable** (structured data) — daily analytics snapshots, metric time series, post metadata. Danny and Kush own the schema and ingestion pipeline.
- **Pinecone Vector DB** (unstructured data) — two namespaces:
  - `surfaced-news` — curated news articles relevant to Christie's brand. Mazen owns ingestion and embedding pipeline.
  - `christies-writing` — Christie's own writing, sourced from the **"Trifecta Right Brain"** Google Drive folder (not scraped from public profiles). Danny and Kush own ingestion.

### Dashboard

The Next.js dashboard reads from both Airtable and Pinecone at runtime. No data is stored in the application layer — the dashboard is a pure read client.

### Repository Structure

All repos live in the **Trifecta-Enterprises** GitHub organization:

| Repo | Purpose | Owner |
|------|---------|-------|
| `tfc-dash` | Dashboard frontend + API routes | Danny & Kush |
| `tfc-brain` | Airtable schema, data ingestion pipelines, scraping jobs | Danny & Kush |
| `tfc-news-agg` | News aggregation + vector DB ingestion | Mazen |

### System Diagram

```
LinkedIn ─┐                                    ┌─────────────────────┐
Ghost ────┤                                    │    TFC Dashboard    │
Substack ─┼──→ tfc-brain (ingestion) ──→ Airtable ──→ Performance Tab │
Medium ───┘                                    │                     │
                                               │                     │
Google Drive ──→ tfc-brain (embedding) ──┐     │                     │
                                         ├──→ Pinecone ──→ Ideas Tab │
News sources ──→ tfc-news-agg ───────────┘     └─────────────────────┘
```

---

## Data Collection Strategy

Platform APIs vary wildly in availability. Our priority order for each platform:

1. **Official API** (Ghost Content API with JWT auth) — use directly where available
2. **Internal API replay** — inspect browser network requests via DevTools, identify the GET endpoints the SPA calls, replay them with appropriate headers
3. **Crawlee** (Node.js scraping framework) — structured scraping with automatic retry, proxy rotation, and rate limiting
4. **Playwright browser automation** — full headless browser for JavaScript-rendered pages, last resort before manual
5. **Manual export** — CSV download from platform dashboards, uploaded to Airtable manually

### Platform Breakdown

| Platform | API Status | Our Approach | Fallback |
|----------|-----------|--------------|----------|
| **Ghost** | Content API fully supported (JWT auth) | Direct API integration | N/A — API is reliable |
| **LinkedIn** | No public analytics API; internal API exists | Inspect network requests, replay GET calls with session cookies | Crawlee scraping, then manual CSV export |
| **Substack** | No official API; unofficial endpoints exist | Internal API replay (reader-facing endpoints) | Crawlee scraping of public newsletter page |
| **Medium** | Official API deprecated (2023); Stats page has internal endpoints | Internal API replay from Stats page | Crawlee scraping, but effort may not be worth it (see Risks) |

---

## Tech Stack

| Layer | Technology | Notes |
|-------|-----------|-------|
| Framework | Next.js 15 (App Router) + TypeScript | Server components + API routes |
| Styling | Tailwind CSS + shadcn/ui + Magic UI | Component library + animated components |
| Data Fetching | TanStack Query | Client-side caching, refetching, optimistic updates |
| Structured Data | Airtable SDK | Read analytics from Airtable base |
| Blog API | Ghost Content API | JWT-authenticated read access |
| Scraping | Crawlee | Node.js scraping framework with anti-detection |
| LLM | Gemini free API | Starting with free tier; swap to Claude API later |
| Vector DB | Pinecone | Mazen owns setup; we consume via SDK |
| Deployment | Vercel | Preview deploys on PR, production on main |

---

## Airtable Schema (Draft)

Single table: **`analytics_daily`**

| Field | Type | Description |
|-------|------|-------------|
| `date` | Date | The date the metric was recorded |
| `platform` | Single Select | `linkedin`, `ghost`, `substack`, `medium` |
| `metric_type` | Single Select | `impressions`, `engagement_rate`, `followers`, `subscribers`, `open_rate`, `click_rate`, `views`, `posts_count` |
| `value` | Number | The metric value for that date |
| `post_title` | Single Line Text | Title of the specific post (optional — only for post-level metrics) |
| `post_url` | URL | Link to the original post (optional) |

**Design rationale:** A single flat table with a `platform + metric_type` compound key makes cross-platform charting trivial — one TanStack Query call filtered by platform, one chart component that maps metric types to series. We avoid the complexity of per-platform tables until the schema proves insufficient.

This schema will be reviewed with Muji before implementation.

---

## Milestones

### M1 — MVP (by June 26)

- Dashboard UI rendering data from Airtable (even if populated with dummy/seed data)
- Ghost Content API integration live and pulling real data
- Two-tab layout functional (Performance + Ideas)
- Deployed on Vercel with preview environments
- Christie has seen and approved a design direction

### M2 — Data Pipeline (by July 24)

- Automated data ingestion working for all four platforms
- Airtable populated with real analytics data on a scheduled basis
- UI polished — responsive, fast, production-quality
- All base dashboard features complete (charts, filters, date ranges, post drill-down)
- Ideas tab consuming Pinecone data (dependent on Mazen's vector DB being ready)

### M3 — AI Features (by August 17)

- RAG chatbot querying Christie's writing via Pinecone
- Content suggestion engine: viral post detection, similar news matching, draft ideas
- Draft post generation with cost guardrails (Gemini free tier limits)
- Content performance prediction based on historical data
- Content calendar with draft archive
- Chat widget + voice interface for conversational interaction

---

## Ownership & Work Split

### Danny (Primary — Architecture & Integration)

- Core dashboard scaffold and deployment pipeline
- All API integrations (Ghost, LinkedIn, Substack, Medium)
- Airtable schema design and ingestion pipeline
- System architecture and data flow design
- Plugin/widget interface so Kush can build modular features independently

### Kush (Features & Frontend)

- M3 AI features (RAG chatbot, content suggestions, draft generation, prediction)
- Frontend polish, animations, responsive refinement
- Modular widget development using the plugin interface Danny provides

### Plugin Architecture

Danny builds the core dashboard with a defined widget interface. Each widget is a self-contained module with:

- Its own data fetching logic
- A standard props contract
- Independent state management

Kush builds feature modules that plug into this interface without touching core dashboard code. This lets both developers work in parallel without merge conflicts.

### External Dependencies

- **Mazen** — owns Pinecone vector DB setup, `surfaced-news` namespace ingestion, and the `tfc-news-agg` repo. Meeting with Muji scheduled for May 27 to finalize vector DB configuration.

---

## Feature Roadmap (M3 Detail)

| Feature | Description | LLM Dependency | Cost Concern |
|---------|-------------|----------------|--------------|
| RAG Chatbot | Query Christie's writing corpus via Pinecone, return contextual answers | Gemini (free tier) | Low — small corpus, infrequent queries |
| Content Suggestion Engine | Detect viral posts, match to similar news, generate content ideas | Gemini (free tier) | Medium — batch processing on schedule |
| Draft Post Generation | Generate full post drafts from topic + context | Gemini / Claude | High — long outputs burn tokens fast |
| Performance Prediction | Predict engagement based on historical patterns + content features | Gemini (free tier) | Low — classification task, short prompts |
| Content Calendar | Archive drafts, schedule content ideas, track pipeline | None (CRUD) | None |
| Chat + Voice Interface | Conversational access to all dashboard features | Gemini / Claude | Medium — depends on session length |

**Cost guardrails:** All LLM calls go through a rate-limited wrapper. Draft generation is gated behind explicit user action (no auto-generation). We start on Gemini's free tier and only upgrade to Claude when the free tier proves insufficient.

---

## Risks

| # | Severity | Risk | Mitigation |
|---|----------|------|------------|
| 1 | CRITICAL | **Medium API is dead** — official API deprecated in 2023, internal endpoints are undocumented and may require authentication tokens that expire | Internal API replay first. If that fails, Crawlee scraping of public stats page. If that fails, deprioritize Medium entirely — it is the lowest-value platform for Christie. |
| 2 | HIGH | **LinkedIn bot detection** — internal API may validate Origin/Referer headers, enforce CSRF tokens, or rate-limit aggressively | Test with minimal request footprint. Use session cookies from a real browser session. If blocked, fall back to manual CSV export on a weekly cadence. |
| 3 | HIGH | **API cost spiral** — auto-draft pipelines and chatbot sessions can burn through LLM credits fast, especially with long-context prompts | Rate limiting wrapper on all LLM calls. Gemini free tier first. Draft generation requires explicit user action. Token budget per session. |
| 4 | MEDIUM | **Substack has no official API** — unofficial endpoints are reverse-engineered and may break without notice | Crawlee scraping as primary fallback. Substack's public newsletter pages are relatively stable HTML. |
| 5 | MEDIUM | **Vector DB dependency on Mazen** — if Pinecone setup is delayed, Ideas tab is blocked | Ideas tab can render a static/placeholder state. Performance tab (Airtable) is fully independent. Mazen meeting is May 27 — we will know the timeline within 48 hours. |

---

## Decisions Log (from Muji Meeting, May 26)

| # | Decision | Rationale |
|---|----------|-----------|
| 1 | **Scraping priority:** Internal API replay first, Crawlee second, Playwright last | Internal APIs are faster, more reliable, and less likely to trigger bot detection than full browser automation. |
| 2 | **Vector DB:** Pinecone (Mazen owns setup, meeting tomorrow to finalize) | Mazen has existing experience with Pinecone. Two-namespace design keeps concerns separated. |
| 3 | **LLM:** Gemini free tier to start, Claude later | Zero cost during development. Claude upgrade only when we hit quality or rate limits. |
| 4 | **UI Design:** Christie decides from 5 proposals Danny will email | Christie is the end user — her preference drives the design direction. |
| 5 | **Airtable:** Danny and Kush design their own base, show Muji for review | We need to own the schema to move fast. Muji reviews for alignment with broader Trifecta data practices. |
| 6 | **Content source for vector DB:** Christie's Google Drive ("Trifecta Right Brain" folder), not scraped from public profiles | Authoritative source, no scraping risk, includes unpublished drafts and internal writing. |
| 7 | **Milestones:** M1 (MVP by June 26) then M2 (pipeline by July 24) then M3 (AI features by Aug 17) | Incremental delivery with review checkpoints. MVP first so Christie has something tangible early. |

---

## Next Steps

- [ ] Email Christie the 5 design proposals
- [ ] Design and implement Airtable schema (seed with dummy data for M1)
- [ ] Scaffold Next.js 15 project with shadcn/ui + TanStack Query + Tailwind
- [ ] Test Ghost Content API integration (JWT auth, pull real posts)
- [ ] Research internal API endpoints for LinkedIn (DevTools network inspection)
- [ ] Wait for Mazen's vector DB decision (Muji meeting him May 27)
- [ ] Upload this document to Google Drive for May 28 standup
