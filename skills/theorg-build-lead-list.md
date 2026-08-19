---
name: Build a lead list from an org chart
description: Find the right people at a target company, resolve their work emails, and save them to a named list on The Org — the agent-only write path.
api: https://developers.theorg.com/mcp
operations: [search_companies, find_positions, resolve_contacts, create_list, add_to_list, get_lists, get_usage]
---

# Build a lead list from an org chart

End-to-end prospecting flow on The Org's MCP surface: identify a company, find the people,
resolve their contact details, and persist them to a list.

**This flow is MCP-only.** `create_list` and `add_to_list` are the only write operations
The Org exposes anywhere — the REST Lists API is read-only (`GET /v1.1/lists`). There is no
way to build a list over REST.

## Auth
- MCP endpoint: `POST https://api.theorg.com/v1.1/mcp`, Streamable HTTP, JSON-RPC 2.0.
- Send `X-Api-Key: <your key>`. Optionally set `MCP-Protocol-Version: 2025-11-25`.
- Handshake: `initialize` → `notifications/initialized` → `tools/list` / `tools/call`.

## Steps
1. **Check budget first.** Call `get_usage` (free) and note `credits` remaining and
   `resetDate`. Steps 3 and 4 are the only paid steps in this flow; everything else is free.
2. **Resolve the company.** Call `search_companies` with a name, domain, email domain, or
   LinkedIn company URL/slug. Free. Use `get_company` (free) if you need size, industry,
   location or funding to qualify the account before spending credits.
3. **Find the people.** Call `find_positions` with filters (job titles, departments,
   org chart levels, locations, employee ranges, `hiredWithinDays`, `verifiedWorkEmail`, …).
   **Costs 1 credit per returned row**, and caps at **25 rows per call**.
   - Narrow the filters before you call. There is no MCP cost estimator — the free
     `POST /v1.1/positions/credit-usage` REST endpoint is the only way to price a query in
     advance, so use REST for estimation if the query is large.
   - Rows returned again within 24 hours cost nothing more.
4. **Resolve contacts.** Call `resolve_contacts` with up to 25 position IDs.
   **1 credit per person newly resolved**; people your account already paid for are free.
   Skip anyone whose `workEmail` already came back populated in step 3.
5. **Create the list.** Call `create_list` with a name and optionally up to 25 position IDs.
   Free.
6. **Add the rest.** Call `add_to_list` with the list ID and up to 25 IDs per call. Free.
   Loop in batches of 25 for larger sets.
7. **Confirm.** Call `get_lists` (free) and check `positionCount`. The list is visible in the
   product at `https://theorg.com/people/lists/{slug}`.

## Walking the reporting line
To expand around a known person rather than filter broadly:
- `find_person` (free) — resolve someone by LinkedIn URL/slug, or full name + company.
- `get_manager` (1 credit per find) — go up one level.
- `get_reports` (1 credit when found, max 50) — go down one level.

## Cost & limits
- Rate limit: 15 requests/second per endpoint across all keys on the account; `429` on
  exceed, with no `Retry-After` header.
- Per-call caps are hard: 25 rows (`find_positions`), 25 people (`resolve_contacts`),
  25 IDs (`add_to_list`, `create_list`), 50 (`get_reports`). Results are truncated at the
  cap — batch rather than expecting a page token, because there is none on the MCP surface.
- Credits are the same balance as REST usage.

## Retry safety — read this before retrying a write
The Org publishes **no idempotency-key contract**. The 24-hour "replay" guarantee is a
*billing* rule (you are not charged twice), not an execution guarantee. If `create_list`
times out, do **not** blindly retry — call `get_lists` first to check whether the list was
created, or you will end up with duplicates. Same for `add_to_list`.

## Errors
Errors come back as `{"error":{"code":<status>,"reason":"<text>"}}`.
- `401` — missing/invalid credentials.
- `402` — credits exhausted. Stop and report the shortfall; retrying cannot succeed.
- `429` — back off with your own interval.
- `500/503/504` — safe to retry reads; for the two writes, verify state first.
