# newsworld-platform

Thin, code-free repo: cross-repo docs, ADRs, the shared API contract, and MCP config for
NewsWorld. See the workspace-root `CLAUDE.md` (sibling folder, one level up if this repo
is cloned inside the `NewsWorld` workspace) for full project context — this repo holds
the durable record of *why*, not application code.

## This repo, specifically

- `adr/` — one file per architecture decision, numbered sequentially. Start every major
  decision here before implementing it; update status (Accepted/Superseded) rather than
  deleting history.
- `openapi/` — the OpenAPI contract shared between `newsworld-backend` and
  `newsworld-frontend`. Still a placeholder as of 2026-11-15; populate once the FastAPI
  service has its first routes.
- `.mcp.json` — reference MCP server config (GitHub, Atlassian/Jira) to copy into the
  other two repos if per-project MCP config is ever needed there; both are currently
  registered at user scope instead.

Do not add application code here — if something needs to run, it belongs in
`newsworld-backend` or `newsworld-frontend`.
