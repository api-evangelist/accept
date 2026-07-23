---
name: Apply a MIDAS control-plane bundle
description: Plan (dry-run) then apply a YAML control-plane bundle of surfaces, profiles, and grants, and review the change audit.
api: openapi/accept-midas-openapi-original.yml
operations: [planBundle, applyBundle, listControlAudit, getSurface, listProfiles]
---

# Apply a MIDAS control-plane bundle

The control plane is a YAML-declared set of governance surfaces, authority profiles, and
grants. Always **plan before apply**.

## Auth
`Authorization: Bearer <token>` with a `platform.operator`/`platform.admin` role. Control-plane
endpoints accept `application/yaml`, `application/x-yaml`, or `text/yaml` bodies and return
`501` when the control plane is not enabled.

## Steps
1. **Plan (dry-run)** — `POST /v1/controlplane/plan` (`planBundle`) with the YAML bundle.
   Review the computed diff before making changes.
2. **Apply** — `POST /v1/controlplane/apply` (`applyBundle`) with the same bundle to commit.
3. **Audit the change** — `GET /v1/controlplane/audit` (`listControlAudit`) to confirm what
   was applied.
4. **Confirm state** — `GET /v1/surfaces/{id}` (`getSurface`) and `GET /v1/profiles`
   (`listProfiles`). New surfaces/profiles move through a review → approve lifecycle before
   they are eligible for evaluation.

## Errors
`{"error": "..."}` envelope. `400` invalid bundle, `403` missing role, `409` conflicting
state, `415` unsupported content-type, `501` control plane not configured.
