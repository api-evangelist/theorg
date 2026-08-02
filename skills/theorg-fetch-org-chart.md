---
name: Fetch a company org chart
description: Retrieve a company's public organizational chart from The Org by domain or LinkedIn URL.
api: https://developers.theorg.com/api/endpoints/company-api
operations: [get_org_chart]
---

# Fetch a company org chart

Use The Org API to pull a company's public org chart as a flat list of ChartNode objects.

## Auth
- Send `X-Api-Key: <your key>` on every request. HTTPS only.
- Create a key from the developer subscription page after signing up at https://theorg.com.

## Steps
1. Identify the company by either its website/email `domain` or its `linkedInUrl`.
2. Call `GET https://api.theorg.com/v1.2/companies/org-chart` with one of:
   - `?domain=example.com`
   - `?linkedInUrl=<company linkedin url or slug>`
   Optionally add `&section=orgChart` (default) or `&section=board`.
3. Read the returned flat list of ChartNode objects (`id`, `jobId`, `jobTitle`, `managerId`,
   `nodeType`, `section`, `positionId`, `fullName`, `title`). Rebuild the tree by linking each
   node's `managerId` to a parent node `id`.

## Cost & limits
- 10 credits per successful request. Check remaining credits with `GET /v1.1/usage`.
- Rate limit: 15 requests/second per endpoint; a `429` means back off and retry.
- Note: v1.2 nodes do NOT include `linkedinUrl` or `workEmail`.

## Errors
Errors come back as `{"error":{"code":<status>,"reason":"<text>"}}`. Handle `429` (rate limit)
and credit-exhaustion cases before retrying.
