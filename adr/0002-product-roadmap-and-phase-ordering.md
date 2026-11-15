# ADR 0002: Product roadmap and phase ordering

Date: 2026-11-15
Status: Accepted
Supersedes: the phase ordering on slide 14 of the November 2026 investor deck

## Context

ADR-0001 recorded the restart decision. This ADR reconciles that decision with the actual
investor deck (NewsWorld_Investor_Pitch_2026.pdf, November 2026) and fixes the phase plan.

The deck's execution slide orders work as: Phase 1 Foundation → Phase 2 Core News →
Phase 3 AI Context → Phase 4 Archives & Community → Phase 5 Nearby & Action. Our restart
plan builds the archive product first. Four decisions were confirmed on 2026-11-15 to
resolve this and other open questions.

## Decisions

**1. Market and corpus: US public domain first (Chronicling America).**
Fastest path, zero licensing risk, and the richest available OCR + word-coordinate data.
Accepted cost: the demo shows early-1900s US newspapers rather than the deck's
"15 March 2015 · Hyderabad edition · The Hindu" example, and the first buyer is a US
library rather than an Indian institution.

**2. Phase ordering: archives first; the investor narrative gets updated.**
This is not a contradiction of the deck. The deck's own Phase 1 instructs us to validate
"content rights" and "target segment". That validation concluded that modern news
licensing is blocked for a pre-revenue team while public-domain archives are open now.
Deck slide 5 already shows the archive experience as the hero product screen, and slide 11
lists Institutions as a served segment. Only the *ordering slide* changes.

**3. Capacity: under 10 engineering hours/week combined, unchanged.**
This is the binding constraint on every estimate below.

**4. No fixed external deadline.** Sequence for soundness, not for demo optics.

## Phase plan

Effort is in engineering hours. At under 10 hrs/week, divide by ~40 for months.

| Phase | Goal | Effort |
|---|---|---|
| **0 — Foundation & Rails** | Able to ship safely: repos, CI, environments, observability | ~20h (~12h done) |
| **1 — The Reading Room** | Deck slide 5's hero screen on public-domain content | ~135h |
| **2 — Cited AI Context** | Deck slide 6's "explain in context", fully grounded | ~45h |
| **3 — Institutional readiness** | Demo-able → sellable; realises slide 12's institutional revenue path | ~60h |
| **4 — Core News (modern)** | The deck's Phase 2. **Gated on a signed feed or licence** | ~120h |
| **5 — Connected Intelligence** | The deck's slide 7, honestly implemented | ~120h |
| **6 — Nearby & Action** | The deck's Local + Action layers. **Separate go/no-go** | ~200h+ |
| **7 — Community & business tools** | Deck Phase 4's "verified community contributions", dashboards, mobile | open |

**Critical path to first revenue: Phases 0–3, ≈260 hours.**

### Exit criteria

- **P1:** a researcher opens `/1910-05-14`, sees the publications of that day, opens a page
  scan, reads OCR beside it, searches within a date/place filter, and shares a permalink
  that still resolves a month later.
- **P2:** selecting a paragraph returns a plain-language explanation whose every claim
  highlights its source region on the scan — and which says "not stated on this page"
  rather than guessing.
- **P3:** a pilot library authenticates by campus IP, its users need no individual
  accounts, and the library receives a monthly usage report.
- **P4:** does not begin until a signed feed or licence exists. This is the rock that
  attempt 1 broke on.
- **P5:** forty reprints of one wire story are reported as **one** independent source,
  with the syndication chain visible.

## Known risks

- **Corpus access changed.** The legacy `chroniclingamerica.loc.gov` API was retired in
  2025; the collection is now reachable only through the **loc.gov API**. LOC documents
  that data across loc.gov services is "not fully synced", and there are third-party
  reports of 403s / rate-limiting. **Validating this source is the first blocking ticket
  of Phase 1**; Phase 1 estimates are not trustworthy until that spike completes.
- OCR quality on early-1900s scans is variable and will affect both search and AI quality.
- Deck slide 13 forbids a "false truth percentage", and slide 7 states that source count is
  not proof of truth. A naive "N sources confirm" badge would violate the deck's own
  principles. Corroboration therefore requires genuine syndication detection (Phase 5).

## Deferred deliberately

Article extraction from page layout · semantic/vector search (pgvector column exists from
day one, index built later) · voice assistant (deck slide 6 — large scope, low early value)
· Local/Nearby (treated as a distinct product bet with its own funding decision, per the
deck's own "monetization only when value is proven").
