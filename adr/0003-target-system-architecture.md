# ADR 0003: Target system architecture and its evolution path

Date: 2026-11-15
Status: Accepted

## Context

The product in the November 2026 deck is a multi-product platform (Knowledge, Local,
Action layers). A two-repo front/back split will not carry it to market. But the failure
mode at a capacity of under 10 engineering hours/week (ADR-0002) is the opposite one:
splitting into services nobody has time to operate.

This ADR therefore records a **target shape plus trigger-based evolution**, not a
big-bang build.

## Logical domains (bounded contexts)

| Domain | Owns | Why it is a boundary |
|---|---|---|
| Content & Rights | `sources`, `editions`, `pages`, OCR, `content_rights` | Product core; the rights gate lives here |
| Ingestion & Enrichment | connectors, jobs, OCR, derivatives, NER, dedup | Bursty, heavy ML deps, must never take down the API |
| Search & Discovery | indexes, facets, ranking | Engine must stay swappable |
| Context & AI | RAG, citation spans, prompt versions, cost accounting | Rate and cost isolation |
| Story Graph | clusters, events, corroboration, provenance | Heavy batch; the deck's differentiator |
| Identity & Entitlements | users, institutions, seats, IP ranges, SAML | Compliance boundary; required in order to sell |
| Local & Places | places, businesses, geo, provider contracts | Different data, buyer and SLA |
| Partner & Commerce | profiles, claims, referrals, billing | Money boundary |
| Platform | audit, config, flags, observability, admin | Cross-cutting |

## Deployment evolution

| Stage | Phase | Deployables | Added infrastructure |
|---|---|---|---|
| A | P0–P2 | `api` (modular monolith, FastAPI) + `ingest-worker` + `web` | Postgres, R2, Vercel; jobs as a Postgres table |
| B | P3 | + `admin` console | Redis (cache, rate limits), SAML/IP auth |
| C | P4–P5 | extract `search`, extract `ai-gateway` | OpenSearch; a broker only if outbox throughput demands one |
| D | P6 | + `places` service | PostGIS, provider integrations |

## The discipline that makes this work

The backend is internally modular **from day one**:

- one module per domain;
- **no cross-module table access** — modules talk through module APIs only;
- domain events written to an **outbox table**.

Extraction into separate services then becomes mechanical rather than a rewrite. This
costs almost nothing today and is the single practice that buys a market-ready
architecture without paying for it now.

## Repository topology

| Repo | Status | Split trigger |
|---|---|---|
| `newsworld-platform` | exists | — |
| `newsworld-backend` | exists | — |
| `newsworld-frontend` | exists | — |
| `newsworld-infra` | add at end of P0 | The moment real cloud resources are provisioned. IaC must not live in an app repo |
| `newsworld-ingest` | add mid-P1 | When OCR/ML dependencies bloat the API image, or worker scaling diverges from API scaling |
| `newsworld-contracts` | add at P3 | When a second client or a third party consumes the API |

Do not exceed six repos. Each additional repo is coordination overhead charged against
ten hours a week.

## Cross-cutting concerns required from day one

These share the character of the `content_rights` decision in ADR-0001: cheap now,
brutal to retrofit.

1. **Rights gate.** One policy function that every content read passes through. Model
   `permitted_uses[]` (`display_thumbnail`, `display_full_scan`, `full_text_display`,
   `ai_derive`, `redistribute`) with effective dates, jurisdiction and required
   attribution text — **not a boolean**. Deck slide 13 permits "original scans or full
   text only where rights allow", and publisher contracts grant display and AI-inference
   rights separately.
2. **Provenance on every displayed item** — source, retrieved-at, rights basis. Slide 9's
   "last-checked date" is this same mechanism surfaced in the UI.
3. **AI audit log** — question, retrieved spans, model, prompt version, answer. Required
   both for the trust claims on slide 13 and for diagnosing hallucination.
4. **Stable permalinks.** Researchers cite them; breaking them destroys institutional
   credibility.
5. **Per-call AI cost accounting** — before the bill, not after.
6. **Word-level OCR coordinates (ALTO/hOCR) captured at ingest.** Without them, the
   slide-5/6 hero feature (citing the exact span on a scan) is impossible, and
   re-ingesting the corpus later is prohibitive.

## Standards adopted

- **IIIF** (Image API + Presentation API) for scan delivery. It is the standard every
  digital library speaks: it provides a mature free viewer ecosystem (OpenSeadragon,
  Mirador), lets institutions embed our content in their own discovery layers, and is an
  immediate credibility signal to librarians. Cheap if adopted at ingest, expensive later.
- **SAML / Shibboleth / OpenAthens + IP-range authentication** (Phase 3). This is how
  libraries actually authenticate; without it the institutional revenue path on slide 12
  is unreachable.
- **WCAG 2.2 AA** (Phase 3). A procurement requirement for public institutions. Our OCR
  text already makes scans screen-reader accessible, which turns a compliance cost into a
  selling point.
