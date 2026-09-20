---
name: piknik-spot-publish-a-community-event
description: Create, update and cancel a farmers-market, workshop or tour event for a place you manage, including weekly recurrence with an iCal RRULE.
api: openapi/piknik-spot-openapi.yml
surface: MCP (https://piknik.spot/api/mcp) or REST under https://piknik.spot/api
operations:
  - get_my_places
  - search_events
  - create_event
  - update_event
  - GET /events
  - POST /events
  - PUT /events/{id}
  - DELETE /events/{id}
  - GET /events/{id}/calendar
  - GET /events/{id}/instances
scopes: [events:write]
method: generated
generated: '2026-09-19'
grounding: >-
  Tool names from the live tools/list (2026-09-19); REST paths and the CreateEventRequest schema
  verbatim from openapi/_original/piknik-spot-openapi.json; scope names from the RFC 8414 document.
---

# Publish a community event

Authenticate as in the marketplace skill, requesting the `events:write` scope.

1. **Resolve the place** with `get_my_places` / `GET /v2/participants/my-businesses` → `participant_id`.
2. **Avoid duplicates:** `search_events` `{ "query", "participant_id", "start_date", "end_date" }` or
   `GET /events?participantId=`.
3. **Create:** MCP `create_event` or REST `POST /events` with `CreateEventRequest`: required
   `participant_id`, `title`, `description`, `start_date`, `end_date` (ISO 8601 date-time), `address`;
   optional `location_name`, `event_type` (`market` | `workshop` | `tour` | `festival` | `meeting` | `other`),
   `is_recurring` (default false), `recurrence_rule` (iCal RRULE, e.g. `FREQ=WEEKLY;BYDAY=SA`),
   `registration_url`. The MCP tool also takes `weekly_days` and `recurrence_end_date`.
4. **Update or cancel:** MCP `update_event` `{ "event_id", ...fields, "status" }` or REST `PUT /events/{id}`;
   `DELETE /events/{id}` removes it. Recurring instances are readable at `GET /events/{id}/instances`
   and the event exports as iCalendar at `GET /events/{id}/calendar`.

Same cautions as the marketplace skill: no idempotency key (re-check step 2 before retrying a POST),
deletion is the documented reversal with no stated window, and public events are indexed once posted.
