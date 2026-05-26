# Access-Control Gadget Spotting Guide

## App Behaviors

- [ ] UI hides an action but the request shape is predictable - possible function-level gate drift.
- [ ] A list response contains IDs for later detail calls - possible metadata to object-read chain.
- [ ] Export/report jobs reuse list filters - possible bulk amplifier.
- [ ] File uploads produce preview, thumbnail, OCR, or signed URL responses - possible derivative ACL drift.
- [ ] GraphQL exposes `node`, `nodes`, global IDs, wide mutation inputs, or nested fields - possible resolver drift.
- [ ] WebSocket messages include object, room, tenant, channel, or action values - possible per-message authorization miss.
- [ ] Responses include cache headers on private-looking API responses - possible cache/CDN context miss.
- [ ] Legacy, mobile, beta, or versioned routes duplicate current features - possible inventory drift.

## Suspicious Parameters

- [ ] `user_id`, `account_id`, `owner_id` - subject/object confusion.
- [ ] `org_id`, `tenant_id`, `workspace_id`, `project_id` - tenant boundary.
- [ ] `parent_id`, `comment_id`, `attachment_id`, `note_id` - child/parent boundary.
- [ ] `role`, `role_id`, `permission`, `scope`, `is_admin` - property or function escalation.
- [ ] `status`, `state`, `approved`, `verified`, `step`, `stage` - workflow state transition.
- [ ] `job_id`, `artifact_id`, `download_url`, `report_id` - async/export amplifier.
- [ ] `channel`, `room`, `topic`, `subscription` - realtime boundary.

## Safe First Questions

- [ ] Does the server derive actor, tenant, and role from the session?
- [ ] Does every object lookup include the current tenant or relationship?
- [ ] Does every child action re-check the parent?
- [ ] Does every field follow role/property rules?
- [ ] Does every bulk item get checked individually?
- [ ] Does every async step check enqueue, execution, status, result, and artifact download?
- [ ] Does every channel enforce the same policy as REST?

