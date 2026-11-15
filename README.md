# newsworld-platform

Cross-repo docs, architecture decisions, the API contract, and shared tooling config for
NewsWorld — a thin repo, not a service. The actual code lives in:

- [newsworld-backend](https://github.com/newsworld-app/newsworld-backend) — FastAPI +
  Postgres API, ingestion, ML
- [newsworld-frontend](https://github.com/newsworld-app/newsworld-frontend) — Next.js +
  TypeScript frontend

## Contents

- `adr/` — Architecture Decision Records, one file per decision
- `openapi/` — OpenAPI contract shared between backend and frontend
- `.mcp.json` — shared Claude Code MCP server config; copy or symlink into each repo

## v1: "The Reading Room"

Institutional archive search over public-domain newspapers (Chronicling America). A
researcher picks a date, sees what was published, opens the page scan, reads OCR text
alongside it, and asks one question answered with citations to the exact span on that
page. See `adr/0001-restart-and-v1-scope.md` for the full restart rationale.

**Binding constraint: under 10 engineering hours/week combined across the team.** Scope
every proposal against this before adding it to the roadmap.

Tracker: Jira Cloud (`NW-` keys).
