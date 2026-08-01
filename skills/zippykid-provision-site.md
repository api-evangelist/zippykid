---
name: Provision a Pressable WordPress site
description: Authenticate, create a managed WordPress site, and confirm it is live.
api: openapi/zippykid-pressable-openapi-original.json
operations:
  - POST /auth/token
  - POST /sites
  - GET /sites/{id}
  - GET /sites
---

# Provision a Pressable WordPress site

Use the Pressable API v1 (`https://my.pressable.com/v1`) to create and verify a managed
WordPress site. The spec ships no `operationId`s, so operations are referenced by
HTTP method + path (all verified present in the OpenAPI).

## Auth
1. `POST /auth/token` with `grant_type=client_credentials`, `client_id`, `client_secret`
   (generated in the My Pressable Control Panel). You receive a Bearer access token valid
   for 1 hour. Send `Authorization: Bearer <token>` on every call. Requires the
   `Sites (Edit)` permission scope on the token.

## Steps
1. `POST /sites` — create the site (form-encoded body). Capture the returned site `id`
   from `data`.
2. `GET /sites/{id}` — poll until provisioning completes and the site reports ready.
3. `GET /sites` — optional: confirm the new site appears in the account list
   (paginate with `page` / `per_page`, max 50).

## Rules
- Requests are form-encoded; responses are the JSON envelope `{ message, data, errors }`.
- No idempotency key is supported — do not blindly retry `POST /sites`; check `GET /sites`
  first to avoid duplicate provisioning.
- Handle errors from the `errors[]` array: 401 (refresh token), 403 (missing scope),
  422 (precondition). See errors/zippykid-problem-types.yml.
- Subscribe to the `site_created` webhook (asyncapi/zippykid-webhooks.yml) for async
  confirmation instead of polling.
