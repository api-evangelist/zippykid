---
name: Connect a repo and deploy to a Pressable site
description: Connect a GitHub repository to a site and trigger a deployment.
api: openapi/zippykid-pressable-openapi-original.json
operations:
  - POST /auth/token
  - POST /sites/{site_id}/git
  - GET /sites/{site_id}/git/branches
  - POST /sites/{site_id}/git/deploy
  - GET /sites/{site_id}/git/history
---

# Connect a repo and deploy to a Pressable site

Wire a GitHub repository to a Pressable-managed WordPress site and ship a deploy.
Operations are referenced by HTTP method + path (verified present in the OpenAPI).

## Auth
1. `POST /auth/token` (client-credentials) for a 1-hour Bearer token. Requires the
   `Git (Edit)` scope.

## Steps
1. `POST /sites/{site_id}/git/token` — store the GitHub access token for the site
   (if not already stored).
2. `POST /sites/{site_id}/git` — connect the repository to the site.
3. `GET /sites/{site_id}/git/branches` — list branches and pick the deploy branch.
4. `POST /sites/{site_id}/git/deploy` — trigger the deployment.
5. `GET /sites/{site_id}/git/history` — confirm the deploy completed.

## Rules
- A site not enrolled in the new GitHub integration returns 422 — resolve the
  precondition in `errors[]` before retrying.
- Prefer the `git_deploy_completed` / `git_deploy_failed` webhooks
  (asyncapi/zippykid-webhooks.yml) over polling history.
- Responses use the `{ message, data, errors }` envelope; auth model in
  authentication/zippykid-authentication.yml.
