---
name: Register webhooks and subscribe to practice events
description: Register an HTTPS webhook endpoint, subscribe to resource events, and verify the HMAC signature on delivered messages.
api: openapi/nexhealth-openapi-original.json
operations:
  - postAuthenticates
  - postWebhookEndpoints
  - postWebhookEndpointsIdWebhookSubscriptions
  - getWebhookEndpoints
---

# Register webhooks and subscribe to practice events

Use this flow to receive event notifications instead of polling.

## Auth and headers
1. **`postAuthenticates`** to get a bearer token; send `Authorization: Bearer <token>` and `Nex-Api-Version: v3.0.0`.

## Steps
1. **Register an endpoint** — `postWebhookEndpoints` (`POST /webhook_endpoints`) with `{ "target_url": "https://your.app/hooks" }`. The URL **must** be HTTPS because payloads may contain PHI. The response returns the endpoint `id` and a `secret_key` — store the secret; you cannot retrieve it later (you would have to recreate the endpoint).
2. **Create a subscription** — `postWebhookEndpointsIdWebhookSubscriptions` (`POST /webhook_endpoints/{id}/webhook_subscriptions?subdomain=...`) with `{ "resource_type": "appointment", "event": "appointment_insertion" }`. `subdomain`, `resource_type`, and `event` are required.
3. **Confirm** — `getWebhookEndpoints` to list your registered endpoints and their active state.

## Verifying delivered events
Each delivery is a `POST` to your `target_url` with headers `timestamp`, `signature`, and `content-type: application/json`. To verify:
1. Base64-encode the raw payload body.
2. Build `signed_payload = "{timestamp}.{base64_payload}"` using the `timestamp` header.
3. Compute `HMAC-SHA256(secret_key, signed_payload)` (hex).
4. Compare it to the `signature` header.

## Reliability
- Respond `2xx` quickly. Failed deliveries retry on a backoff schedule out to 48h (30s, 1.5m, 3.5m, 10m, 30m, 2h, 5h, 10h, 24h, 48h).
- After 48h of continuous failures the endpoint's `active` flag is set to false and deliveries stop.
