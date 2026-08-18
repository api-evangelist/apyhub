# ApyHub — re-profile findings, 2026-08-18

Triggered by Nikolas Dimitroulakis (Co-Founder, ApyHub) asking why APIs.io listed
only 4 ApyHub APIs when the catalog holds 450+ services and 1,500+ endpoints.

He is right. The listing was wrong, and the cause was on our side.

## Why it was 4

The ApyHub profile was never harvested. An early enrichment pass **scaffolded** a
single 5-path `apyhub-openapi.yaml` from ApyHub's marketing copy, then split it by
tag into four "APIs" — Convert, Currency, Extract, Generate. Those four scaffolds
are what APIs.io indexed, scored, and displayed.

We fetched ApyHub's `llms.txt` on 2026-08-17 but never rebuilt the profile from it,
so the harvest sat unused next to a fabricated contract set.

The four scaffolds and their parent are moved to `_quarantine/`. They are not
ApyHub's definitions and were never published by ApyHub.

## What is actually there

Harvested from `https://apyhub.com/llms.txt` and the per-service `.md` documentation
ApyHub publishes at every catalog URL:

| Measure | Count |
| --- | ---: |
| Services documented | 451 |
| Listed providers | 19 |
| Categories | 21 |
| Endpoint rows documented | 987 |
| Endpoint rows with a path published | 792 |
| Endpoint rows with **no** path published | 195 |
| Unique method + path pairs | 728 |
| **Verified callable** (401 on `api.apyhub.com`) | **552** |
| Paths that 404 (fragment, not absolute) | 240 |
| Services with ≥1 verified callable endpoint | 210 |
| Services with no resolvable path at all | 241 |

Verification method: each published path was issued unauthenticated against
`https://api.apyhub.com`. A real path returns `401 {"error":"missing api key"}`;
a nonsense path returns `404 {"error":"not found"}`. The host discriminates
correctly, so 401 is positive evidence the operation exists and is gated.

## What ApyHub does well

- `llms.txt` (109 KB) and `llms-full.txt` (742 KB), first-party and current.
- Content negotiation: every page is available as Markdown via `.md` or
  `Accept: text/markdown`. Rare and genuinely useful.
- A live MCP server at `https://mcp.eu.apyhub.com` that is **RFC 9728 compliant** —
  401 carries `WWW-Authenticate: Bearer resource_metadata="..."`, and
  `/.well-known/oauth-protected-resource` resolves with `authorization_servers`,
  `bearer_methods_supported`, `resource` and `scopes_supported`. Most providers
  claiming an MCP server do not get this right.
- Per-call cost published as `atoms` on nearly every operation.

## Gaps on ApyHub's side

These are why even a correct harvest cannot reach 451 callable contracts.

1. **No OpenAPI anywhere.** `/openapi.json`, `/.well-known/openapi.json` and
   `api.apyhub.com/openapi.json` all 404. For a certification-led marketplace this
   is the single highest-value fix.
2. **`llms-full.txt` does not contain endpoints.** It advertises itself as "every
   catalog service with its endpoints, inlined" — 451 service records are present
   and **zero** `## Endpoints` sections are. The per-service `.md` pages have them;
   the aggregate file drops them.
3. **195 endpoint rows publish an empty path** (`` ` ` ``). This is concentrated
   almost entirely in marketplace-provider namespaces — Dosvak 0/154 services
   resolvable, nick creighton 0/21, Quadlem 0/11, FlowDocs 0/7, SE Ranking 0/6.
   The page generator appears to emit an empty Path cell for third-party listings.
4. **240 paths are fragments, not absolute paths** — `/extract`, `/link`,
   `/download`, `/file`, `/split` — missing their service prefix, so they 404.
5. **The base URL is never stated.** No service page names `api.apyhub.com` and
   there is not a single `curl` example across all 451 pages. A reader of the docs
   cannot construct a call without already knowing the host.
6. `/docs` returns 404 (we had it recorded as the documentation URL for all four
   APIs — now corrected to `/quickstart`).
7. Path parameters use `:jobId` rather than `{jobId}`.

Reconciliation: ApyHub states 1,500+ endpoints. Their public machine-readable
surface documents 987 and makes 552 verifiable. The remaining ~500 are not
discoverable by any agent or catalog from the outside.

## What was rebuilt

- `apis.yml` — 451 API entries, real names, descriptions, categories, per-service
  documentation URLs, listed-provider attribution, and an `x-contract-status` of
  `verified` or `no-published-path` on every entry. Provider-level properties now
  record llms.txt, llms-full.txt, sitemap, MCP, pricing, terms and privacy.
- `openapi/` — 210 OpenAPI 3.1 definitions covering the 552 verified operations,
  one per service, carrying path, method, description, tags, security scheme
  (`apy-token` header) and `x-apyhub-atoms` cost. Every file is headed with its
  derivation and verification provenance.
- **Request/response schemas are deliberately omitted.** ApyHub does not publish
  them. Inventing them is what produced the four scaffolds this pass removed.
- `agentic-access/` — regenerated against the 552 verified operations
  (506 acting/write, 46 retrieving/read), replacing output derived from the scaffold.
- `llms/` — llms.txt and llms-full.txt captured.

## Open defects on our side

- **Kin Score regulatory misclassification.** ApyHub scored against
  `energy_utilities` (CDR Energy, Green Button, Smart Energy Code) because its
  `utility` tag matches that regime's trigger list. ApyHub ships developer utility
  APIs, not energy utilities. Same class of bug as the university/government
  collision fixed in 0.12. The new tag set still contains "Utility APIs", so this
  will re-trigger until the rubric adds a guard.
- **`mcp_server: false`** in the 2026-08-17 Agent Readiness run, against a live,
  RFC 9728-compliant MCP server. Under-credited.
- Score cannot be refreshed until APIs.io rebuilds — the scorer reads built
  provider pages, not this repo.

## Remaining pipeline steps

1. Human review of this diff (224 files).
2. Commit + push `all/apyhub` (CLI only).
3. APIs.io rebuild — regenerates the provider page and 451 API pages.
4. Re-score.
