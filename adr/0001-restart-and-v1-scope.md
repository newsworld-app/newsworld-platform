# ADR 0001: Restart NewsWorld, narrow v1 to institutional archive search

Date: 2026-11-15
Status: Accepted

## Context

A 2-person prototype built Mar–Apr 2026 (Sprint 1/2) had drifted from the Nov 2026
investor deck: it modeled a consumer feed with community uploads, points, and vouchers,
had no `editions`/city-edition concept (the deck's hero archive feature), and carried
real security issues (no JWT verification anywhere, client-trusted `userId` on sensitive
routes, a hardcoded JWT secret, an endpoint leaking user emails via sequential IDs).
Licensing modern publisher content was also an unresolved blocker.

## Decision

Restart from scratch with a narrower wedge and a public-domain-first corpus:

- **Wedge:** institutional archive search (libraries, universities, research orgs) —
  known buyers, known licensing path, near-zero CAC. Not the consumer feed, not the
  Local/Nearby layer.
- **Corpus:** public domain first (Chronicling America, Internet Archive, Europeana
  Newspapers, pre-1964 Indian material), so the product can be built and demoed before
  any publisher licensing deal exists.
- **Stack:** all Python/FastAPI (API + ingestion + ML) + Next.js/TypeScript frontend.
  Postgres 16 alone (tsvector FTS, pgvector, later PostGIS) — no Kafka/Temporal/
  OpenSearch/EKS at this stage.
- **Hosting:** managed PaaS, ~$0–40/mo (Neon/Supabase, Cloudflare R2, Railway/Render,
  Vercel).
- **Repos:** kept the two existing GitHub repos under `newsworld-app`, old code preserved
  on `archive/sprint-1`, `main` reset to a clean slate. Added this `newsworld-platform`
  repo for cross-cutting docs.
- **Non-negotiable even in v1:** a `content_rights` record on every item with a rights
  filter on every read path, and real `sources`/`editions`/`pages` entities — cheap now,
  brutal to retrofit.

v1, "The Reading Room": date → publication list → page scan + OCR reader → one cited
AI explain-passage feature. Chronicling America only, 3–5 publications, one date range,
~50k pages. Estimated ~230 engineering hours (~6 months) at the team's real capacity of
under 10 hours/week combined.

Deliberately deferred: article extraction from page layout, story clustering/
corroboration, semantic/vector search (column present, not indexed), user accounts,
Local/Nearby + Action layer + mobile + points/vouchers.

## Consequences

- Any proposal to add scope must be checked against the <10 hrs/week capacity ceiling
  and the deferred-features list before being accepted.
- The old Sprint-1 code and its security issues are not to be ported forward without a
  deliberate, separate decision.
