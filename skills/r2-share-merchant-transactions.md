---
name: Onboard a merchant and share transaction data
description: Register/process a merchant on R2 and push their sales transactions so R2 can score them for financing.
api: openapi/r2-openapi.json
operations:
- "PATCH /webhooks/merchants/{external_id}"
- "POST /transactions/"
- "POST /transactions/async"
- "GET /financings/"
---

# Onboard a merchant and share transaction data

Use this flow to make one of your platform's merchants known to R2 and feed the
transaction history R2 uses to underwrite financing offers.

## Auth
Sign a JWT with your R2-issued `keyId` (as the `kid` header) and `JWTSecret`
using HS256. Include the merchant id as the `mid` claim and, for multi-country
partners, the ISO-2 `country` claim. Send it as `Authorization: <jwt>`.
See `authentication/r2-authentication.yml`.

## Steps
1. **Process the merchant** — `PATCH /webhooks/merchants/{external_id}` with the
   merchant's `external_id` (and `api_key` query param) to create/update the
   merchant record on R2.
2. **Share transactions** — `POST /transactions/` with the merchant's sales
   transactions. Each request may only contain transactions for **one** financing.
   For large batches use `POST /transactions/async`.
3. **Check for offers/financings** — `GET /financings/` (supports `page`,
   `per_page`, `sort`) to see financings issued to the merchant.

## Rules
- One financing per create request (transactions and collections).
- Errors surface as HTTP 400/404/500 with a JSON `{error}` envelope — there is no
  `application/problem+json`. See `errors/r2-problem-types.yml`.
- No idempotency key is supported; do not blindly retry writes.
