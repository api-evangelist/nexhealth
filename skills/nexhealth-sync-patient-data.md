---
name: Sync patients and appointments from a practice
description: Authenticate and page through a practice's institutions, locations, patients, and appointments to build or refresh a local mirror.
api: openapi/nexhealth-openapi-original.json
operations:
  - postAuthenticates
  - getInstitutions
  - getLocations
  - getPatients
  - getPatientsId
  - getAppointments
---

# Sync patients and appointments from a practice

Use this flow to pull a practice's roster and schedule for an incremental sync.

## Auth and headers
1. **`postAuthenticates`** to get a bearer token; send `Authorization: Bearer <token>` and `Nex-Api-Version: v3.0.0` on every request.

## Steps
1. **Discover scope** — `getInstitutions` then `getLocations` to enumerate the `institution_id` / `location_id` values you have access to.
2. **List patients** — `getPatients` (`GET /patients`) scoped by `subdomain` + `location_id`; page with `page`/`per_page`.
3. **Fetch a single patient** — `getPatientsId` when you need the full record by NexHealth id.
4. **List appointments** — `getAppointments` (`GET /appointments`); provide at least one of `location_id` or `foreign_id`, and a start-time window. Appointments support cursor pagination.

## Incremental sync
- Prefer `updated_since` filters where offered so you only pull changed records.
- For high-volume resources (Appointments, Procedures, Charges, Payments, Claims, Adjustments) use **cursor pagination**: read `page_info.end_cursor` and pass it as `end_cursor` on the next call until `has_next_page` is false. `per_page` defaults to 5, max 1000.
- To avoid polling, subscribe to webhooks instead (see `nexhealth-setup-webhooks.md`).

## Conventions and errors
- Envelope `{code, data, description, count, error}`; `count` is the total for the collection, not the page length.
- `foreign_id` + `foreign_id_type` link each record to the source EHR/PMS.
- Handle `429` with backoff; a `403`/`404` on an id usually means it belongs to a location your token cannot access.
