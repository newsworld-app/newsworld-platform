# ADR 0004: Delivery model, real capacity, and corpus access findings

Date: 2026-11-16
Status: Accepted
Amends: ADR-0002 (capacity assumptions), ADR-0003 (frontend delivery)

## Context

Four constraints were clarified on 2026-11-16 that materially change ADR-0002's estimates,
and research into the Chronicling America corpus produced findings that change how
Phase 1 ingestion must be built.

## Decisions

**1. Delivery: responsive web / PWA first. Native mobile apps deferred.**

Mobile use was stated as a priority. Native apps were nonetheless rejected for v1:

- one codebase instead of two, which matters enormously at this capacity;
- installable to a phone home screen, so the mobile requirement is substantially met;
- no app-store developer accounts (an Apple account is $99/year — real money against a
  near-zero budget);
- v1's wedge is institutional archive search, whose users are on desktops.

The unresolved problem is genuine and is tracked as a design spike (NW-27): a 1910
broadsheet is roughly 15 x 22 inches with six to eight dense columns, and panning that on
a 6-inch screen is a poor experience. The likely answer is that mobile defaults to
reflowed OCR text with the scan as a secondary view, inverting the desktop layout.

**2. Inference: free-tier hosted API, behind a pluggable interface.**

The team has near-zero budget. The interface matters more than the provider: free tiers
change terms, throttle, and disappear, while the retrieval and citation plumbing is the
durable, valuable part. The product must degrade gracefully when quota is exhausted — a
demo must never die because a free tier ran out.

**3. Real development capacity is ONE person.**

ADR-0002 recorded "under 10 engineering hours/week combined across both members". This was
misleading. The second team member is design-focused and will not be implementing. The
correct reading is: **under 10 hours a week of a single developer's time**, with design
proceeding in parallel but not reducing the implementation load.

Consequence: the ~260h critical path to Phases 0–3 is roughly **seven months of
wall-clock**, not the shorter figure implied by "combined". Design tickets are tracked
separately (labelled `design`) so the two work streams are not confused in planning.

**4. Corpus sized to fit free tiers.**

Rather than ADR-0002's assumed ~50,000 pages, size v1 to roughly **5,000 pages**, which at
about 1MB of derivatives per page fits within Cloudflare R2's free tier. A smaller corpus
that demos well and costs nothing beats a large one that cannot be afforded. Revisit after
the ingestion spike produces real file sizes.

**5. Develop locally; provision cloud at the END of Phase 1.**

Paying for idle infrastructure while developing against a local Postgres is waste at this
budget. Provisioning is scheduled as the last Phase 1 ticket (NW-33), not the first.

## Corpus access findings (researched 2026-11-16)

**Confirmed:**
- Five file types exist per page: PDF, **JP2**, **JPG**, **OCR text**, and **ALTO XML**.
  ALTO carries the word-level coordinates the hero feature depends on.
- A Datasets portal offers bulk OCR downloads across 3,000+ batches and 23M+ pages, with
  JSON/CSV/TXT listings.
- Documented rate limits: **JSON/YAML API 20 requests/minute** (1-hour block if exceeded);
  text and image services 150/minute. No API key required.

**Open risk:** programmatic requests to `www.loc.gov` from a development environment
returned **HTTP 403** on 2026-11-16, while the same pages load in a browser. LOC documents
429 for rate limiting, not 403, so this is likely a WAF, datacentre-IP or geographic block.
**It must be re-tested from the machine that will actually run ingestion** (NW-9).

**Architectural consequence:** prefer **bulk batch download over API crawling**. At 20
requests/minute, fetching 50,000 pages page-by-page is roughly 40 hours of wall-clock
crawling. Bulk batches avoid this entirely.

**Principle:** do not attempt to evade any block — no user-agent spoofing, no proxies.
Deck slide 13 makes "no unauthorized scraping" a non-negotiable product principle, and our
ingestion must be defensible to the very institutions we eventually want as customers. If
access is genuinely blocked, contact LOC through official channels.

## Correction to ADR-0001

ADR-0001 recorded "`articles` partitioned by `published_date` from day one". This is wrong
on two counts: there is no `articles` table in the v1 schema (it is
`sources`/`editions`/`pages`), and at v1 corpus size partitioning is premature
optimisation that adds real complexity for no benefit.

**Revised:** design tables so they *can* be partitioned later — keep the date column, avoid
cross-partition unique constraints — but do not partition now.
