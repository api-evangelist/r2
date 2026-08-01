---
name: Register and handle R2 event callbacks (webhooks)
description: Register a callback URL and correctly verify, acknowledge, and de-duplicate R2 financing/collection events.
api: openapi/r2-openapi.json
operations:
- "POST /callbacks/"
- "GET /callbacks/"
- "PUT /callbacks/{id}"
- "GET /events/{id}"
---

# Register and handle R2 event callbacks (webhooks)

R2 pushes real-time events (financing_created, financing_paid, collection_returned,
etc.) to a callback URL as HTTP POSTs. See `asyncapi/r2-events-webhooks.yml`.

## Steps
1. **Register a callback** — `POST /callbacks/` (or `PUT /callbacks/{id}` to
   update) with your HTTPS `url`. List existing callbacks with `GET /callbacks/`.
2. **Verify the signature** — each request carries `Content-Sha256`, a base64
   `HMAC-SHA256(payload, jwtSecret)`. Recompute and compare before trusting it.
   Requests originate from R2's fixed IPs (prod `54.215.38.39`, dev `54.151.119.16`).
3. **Acknowledge** — return HTTP `200` with the exact body `R2 API Event Received`.
   Anything else counts as a failure.
4. **De-duplicate** — R2 guarantees delivery, not exactly-once, and retries up to
   6 times over ~30h. You are responsible for dropping duplicate `event` ids.
5. **Fetch detail** — `GET /events/{id}` to re-read a specific event.

## Rules
- Branch on `event.event_type`; the payload includes a `financing`/`collection`
  object depending on the event.
- Respond within 30s or the delivery is treated as failed.
