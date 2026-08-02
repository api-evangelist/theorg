---
name: Prospect positions
description: Search The Org for people/positions matching company, department, title, location, and size filters, estimating credit cost first.
api: https://developers.theorg.com/api/endpoints/position-api
operations: [find_positions, get_usage]
---

# Prospect positions

Search The Org's position graph for people matching a set of filters, with a free
cost estimate before you spend credits.

## Auth
- Send `X-Api-Key: <your key>` on every request. HTTPS only.

## Steps
1. Build a `filters` object (company domains, departments, job titles, locations, industries,
   employee ranges, etc.) plus required `limit` (max 1000) and `offset` (max 10000).
2. Estimate cost first (free): `POST https://api.theorg.com/v1.1/positions/credit-usage`
   with the same JSON body. This returns the credit cost without spending.
3. Confirm you have budget: `GET https://api.theorg.com/v1.1/usage`.
4. Run the search: `POST https://api.theorg.com/v1.1/positions` with the JSON body.
   Each returned Position row costs 1 credit.
5. Read Position fields — `id`, `name`, `title`, `workEmail`, `directDial`, `linkedInUrl`,
   `currentCompany`, `location`, `managerIds`, `reportIds`, etc.
6. Page by advancing `offset`.

## Cost & limits
- 1 credit per returned row. Rate limit: 15 requests/second per endpoint (`429` on exceed).
- Identical requests can be replayed free within 24 hours of the first call.

## Errors
Errors come back as `{"error":{"code":<status>,"reason":"<text>"}}`.
