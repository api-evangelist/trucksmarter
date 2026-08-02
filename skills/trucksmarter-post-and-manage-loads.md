---
name: Post and manage loads on TruckSmarter
description: >-
  Post, update, and remove freight loads on the TruckSmarter load board via
  the partner Load Posting API.
api: openapi/trucksmarter-load-posting-openapi.yml
operations:
  - postLoads
  - deleteLoads
generated: '2026-07-21'
method: generated
source: openapi/trucksmarter-load-posting-openapi.yml
---

# Post and manage loads on TruckSmarter

Use the TruckSmarter Load Posting API (base URL `https://api.trucksmarter.com`)
to post freight loads to the TruckSmarter load board and remove them when they
are covered or cancelled.

## Auth

Every request requires a partner API key as a Bearer token:

```
Authorization: Bearer YOUR_API_KEY
```

Keys are issued by TruckSmarter to partners — there is no self-serve signup
for API keys. Never log or echo the key.

## Steps

1. **Post or update loads** — `postLoads` (`POST /loads/postings/post/`) with a
   JSON body `{ "loads": [ ... ] }`.
   - Each load requires `load_id` (unique string), `equipment[]` (from the
     19-value equipment enum, e.g. `van`, `reefer`, `flatbed`), and `stops[]`.
   - Provide `contact_phone` or `contact_email` — at least one is required.
   - Each stop requires `start_time` (ISO 8601), `city`, `state`, `index`
     (0 = first stop), `type` (`P` pickup or `D` delivery), `latitude`,
     `longitude`, and `country` (ISO 3166-1 alpha-2).
   - Optional load fields: `weight` (lbs), `length` (ft), `distance` (mi),
     `offer_price` (USD), `commodity`, `requirements[]` (24-value enum, e.g.
     `hazmat`, `team`, `tarps`), `general_notes`.
2. **Verify the response** — a 200 returns
   `{ "success": true, "loadCount": n, "message": "n load posted successfully" }`.
   A 400 means the body failed validation; re-check required fields and enum
   values (no error body schema is published).
3. **Update a load** — re-send `postLoads` with the same `load_id`; the
   operation is a documented create-or-update, so replays with the same
   `load_id` update rather than duplicate.
4. **Remove loads** — `deleteLoads` (`POST /loads/postings/delete/`) with
   `{ "loadIds": ["...", "..."] }` once loads are covered or cancelled. A 200
   returns `{ "success": true, "removeCount": n, "message": "n loads removed" }`.

## Rules

- Batch where possible: both operations accept arrays.
- Keep `load_id` stable across updates — it is the idempotency key.
- No pagination, rate-limit, or webhook surface is published; poll nothing —
  this API is write-only from the partner side.
