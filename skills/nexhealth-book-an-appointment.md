---
name: Book a patient appointment
description: Authenticate, find bookable availability, create or match the patient, and book an appointment through the NexHealth Synchronizer API.
api: openapi/nexhealth-openapi-original.json
operations:
  - postAuthenticates
  - getLocations
  - getProviders
  - getAppointmentTypes
  - getAvailableSlots
  - postPatients
  - postAppointments
---

# Book a patient appointment

Use this flow to book an appointment for a patient at a NexHealth-synced practice.

## Auth and headers
1. Exchange your API key for a bearer token: **`postAuthenticates`** (`POST /authenticates`) with header `Authorization: <API_KEY>`.
2. On every request send `Authorization: Bearer <token>` and `Nex-Api-Version: v3.0.0`. Tokens last 1h (production) / 24h (sandbox); re-authenticate when you get a `401`.
3. Scope every call with the practice `subdomain` and a `location_id`.

## Steps
1. **Resolve the location** — `getLocations` to confirm the `location_id` and its `institution_id`.
2. **Pick a provider** — `getProviders` for that institution.
3. **Pick an appointment type** — `getAppointmentTypes`.
4. **Find availability** — `getAvailableSlots` (`GET /available_slots`); you must supply at least one of `pids` (provider ids) or `appointment_type_id`, plus the date range and location(s).
5. **Create or match the patient** — `postPatients` (`POST /patients`). Set `return_existing_if_match: true` to get the existing patient (HTTP 200) instead of a duplicate; a new patient returns 201.
6. **Book** — `postAppointments` (`POST /appointments`) with the patient, provider, location, appointment type, and a slot start time. Set `is_guardian: true` when a guardian books for a dependent; set `unavailable: true` to create a block instead of a patient appointment.

## Conventions and errors
- Responses use the envelope `{code, data, description, count, error}`; check `code` and inspect `error[]` even on 2xx (non-fatal validation messages can appear there).
- Handle `429` (rate limit: 1000 req/min on patients/appointments) with sleep-and-retry.
- Write-backs to the practice's EHR/PMS are asynchronous; the returned record may lag the source system.
