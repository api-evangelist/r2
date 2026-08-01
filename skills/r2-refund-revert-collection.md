---
name: Refund or revert a collection
description: Reconcile a repayment by partially refunding or fully reverting a collection against a financing.
api: openapi/r2-openapi.json
operations:
- "GET /collections/"
- "GET /collections/{id}"
- "POST /api/payments/payin/refund/{id}"
- "POST /api/payments/payin/revert/{id}"
---

# Refund or revert a collection

Use this flow to correct a repayment collection on a financing.

## Steps
1. **Find the collection** — `GET /collections/` (filter by `financing_id`,
   `status`, `kind`, `partner_collection_id`; paginate with `page`/`per_page`/
   `sort`) or `GET /collections/{id}` to fetch one by R2 collection id.
2. **Partial refund** — `POST /api/payments/payin/refund/{id}` with the **R2
   collection id** to refund part of a collection.
3. **Full revert** — `POST /api/payments/payin/revert/{id}` with the **R2
   collection id** to reverse a collection entirely.
4. **Confirm via events** — R2 emits `collection_returned` and may set the
   financing back to `financing_active` after a revert/refund. Handle these in
   your callback (see the events-callbacks skill).

## Rules
- These endpoints expect the **R2** collection id, not your `partner_collection_id`.
- Errors surface as HTTP 400/404/500 with a JSON `{error}` envelope.
