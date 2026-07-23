---
name: Evaluate an agent decision with MIDAS
description: Ask MIDAS whether an agent is within authority to act, then handle the single outcome and its audit envelope — including escalation review.
api: openapi/accept-midas-openapi-original.yml
operations: [evaluate, getDecisionByRequestId, getEnvelope, listEscalations, createReview]
---

# Evaluate an agent decision with MIDAS

MIDAS returns **exactly one outcome** per evaluation — `accept`, `escalate`, `reject`, or
`request_clarification` — plus one tamper-evident audit `Envelope`.

## Auth
Send `Authorization: Bearer <token>` on every `/v1/*` call (tokens are configured via
`MIDAS_AUTH_TOKENS`). In local dev, `MIDAS_AUTH_MODE=open` disables auth.

## Steps
1. **Evaluate** — `POST /v1/evaluate` (`evaluate`). Body includes `surface_id`, `agent_id`,
   `consequence`, `confidence`, `context`, and `process_id` (required in enforced structural
   mode). Set `request_id` as your **idempotency key**: replaying the same
   `(request_source, request_id)` returns the same decision; a differing payload on the same
   pair returns `409`.
2. **Branch on `outcome`**:
   - `accept` → proceed with the action.
   - `reject` → do not act; surface the `reason_code` (e.g. `POLICY_DENY`,
     `CONSEQUENCE_EXCEEDS_LIMIT`).
   - `escalate` → the envelope enters `AWAITING_REVIEW`; wait for a human review resolution.
   - `request_clarification` → gather the missing context keys and re-evaluate.
3. **Look up a prior decision** — `GET /v1/decisions/request/{requestId}`
   (`getDecisionByRequestId`) to retrieve the recorded outcome idempotently.
4. **Inspect the envelope** — `GET /v1/envelopes/{id}` (`getEnvelope`).
5. **Resolve escalations** (reviewer) — list with `GET /v1/escalations` (`listEscalations`),
   then `POST /v1/reviews` (`createReview`) with `APPROVED`/`REJECTED`.

## Errors
Errors return `{"error": "..."}`. Handle `400` (validation), `403` (role), `404` (unknown id),
`409` (idempotency conflict / wrong state). Authority outcomes are `200`, never error codes.
