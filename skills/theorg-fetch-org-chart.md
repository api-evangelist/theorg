---
name: Fetch a company org chart
description: Retrieve a company's public organizational chart from The Org by domain or LinkedIn URL, choosing correctly between the REST data payload and the MCP embed URL.
api: https://developers.theorg.com/api/endpoints/company-api
operations: [GET /v1.2/companies/org-chart, get_org_chart, get_usage]
---

# Fetch a company org chart

Pull a company's public org chart from The Org. **Pick your surface first** — since
2026-08-13 the REST operation and the MCP tool of the same name return different things.

| You want | Use | Cost |
|---|---|---|
| Org chart **data** (nodes, reporting lines, IDs you can traverse) | `GET /v1.2/companies/org-chart` | 10 credits |
| A **rendering** of the chart to show a user | MCP tool `get_org_chart` | free |

Do not assume the MCP tool returns JSON nodes — it returns a signed iframe embed URL
(`/embeds/org-chart/{signature}`). If you need to walk the hierarchy, you must use REST.

## Auth
- Send `X-Api-Key: <your key>` on every request. HTTPS only — plain HTTP fails.
- Create a key at https://theorg.com/subscription#api after signing up.
- The same key authenticates REST and the MCP endpoint.

## Steps (REST — org chart data)
1. Identify the company by either its website/email `domain` or its `linkedInUrl`.
2. Call `GET https://api.theorg.com/v1.2/companies/org-chart` with one of:
   - `?domain=example.com`
   - `?linkedInUrl=<company linkedin url or slug>`
   Optionally add `&section=orgChart` (default) or `&section=board`.
3. Read the returned flat list of ChartNode objects (`id`, `jobId`, `jobTitle`, `managerId`,
   `nodeType`, `section`). Rebuild the tree by linking each node's `managerId` to a parent
   node `id`. Root nodes have no `managerId`.
4. To go from a node to a full person record, use its `positionId` against the position
   search, or call `get_manager` / `get_reports` to walk the line directly.

## Steps (MCP — embeddable chart)
1. Call `get_org_chart` with the company identifier.
2. Render the returned embed URL in an iframe. It is free and consumes no credits.

## Cost & limits
- REST: 10 credits per successful request. Check remaining credits with `GET /v1.1/usage`
  (free) before a batch.
- Rate limit: 15 requests/second per endpoint, counted across every key on the account.
  A `429` means back off — **no `Retry-After` header is returned**, so choose your own
  interval (start at 1s and grow).
- Replaying the same request within 24 hours costs no additional credits, so retries after
  a timeout are safe on the billing side.
- v1.2 nodes do **not** include `linkedinUrl` or `workEmail`. If you need contact data, use
  position search or `resolve_contacts`.

## Errors
Errors come back as `{"error":{"code":<status>,"reason":"<text>"}}` — there is no error
type or field-level detail, only prose in `reason`.
- `401` — missing/invalid key.
- `402` — out of credits. Stop; do not retry, top up or wait for the reset date from
  `GET /v1.1/usage`.
- `404` — company not indexed by that domain/LinkedIn URL. Try the other identifier before
  concluding it is absent.
- `429` — rate limited. Back off.
- `500/503/504` — retry with backoff; the read is safe to repeat and costs nothing extra
  within the 24h replay window.
