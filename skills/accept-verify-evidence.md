---
name: Verify MIDAS decision evidence
description: Retrieve and verify the tamper-evident evidence for a MIDAS decision envelope — integrity, audit events, and the full evidence packet.
api: openapi/accept-midas-openapi-original.yml
operations: [getEvidenceEnvelopeIntegrity, getEvidenceEnvelopeAuditEvents, getEvidenceEnvelopePacket, searchEvidenceAuditEvents]
---

# Verify MIDAS decision evidence

Every evaluation is recorded in a hash-chained, tamper-evident envelope. The Evidence API is
read-only and stateless — the integrity verifier can be re-run idempotently.

## Auth
`Authorization: Bearer <token>` on all calls. The Evidence API returns `501` when the
evidence/introspection service is not configured.

## Steps
1. **Verify integrity** — `GET /v1/evidence/envelopes/{id}/integrity`
   (`getEvidenceEnvelopeIntegrity`). Confirms the hash chain is intact.
2. **Read the audit events** — `GET /v1/evidence/envelopes/{id}/audit-events`
   (`getEvidenceEnvelopeAuditEvents`). Paginate with the opaque `cursor` + `limit` + `order`;
   pass `next_cursor` back verbatim (a `desc` cursor replayed against `asc` returns `400`).
3. **Export the packet** — `GET /v1/evidence/envelopes/{id}/packet`
   (`getEvidenceEnvelopePacket`) for the complete, portable evidence bundle.
4. **Search across envelopes** — `GET /v1/evidence/audit-events`
   (`searchEvidenceAuditEvents`) with the same cursor pagination.

## Errors
`{"error": "..."}` envelope. `404` unknown envelope, `501` service not configured, `400`
malformed cursor/limit.
