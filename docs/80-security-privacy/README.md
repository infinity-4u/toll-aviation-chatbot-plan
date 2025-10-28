```md
# Security, Privacy, Safety

## Privacy & PII
- **Data minimisation**: collect only what the form needs (First/Last, Email, Phone, Company, Reason, Message, Consent).
- **Consent**: explicit privacy + contact consent before creating a lead.
- **Retention**: default 30 days for raw chat + contact payloads unless extended by policy.
- **Redaction**: remove PII from logs/telemetry; never include PII in prompts.

## Security
- **Transport & storage**: TLS in transit; encrypted-at-rest for all stores (Neo4j, Redis, Postgres if used).
- **Secrets**: store in cloud Secrets Manager/KMS; rotate; never commit to Git.
- **Access control**: least-privilege service accounts for Calendar/Email/CRM; RBAC on infra; audited actions.
- **Auditability**: structured logs for tool calls (who/what/when), without PII content.

## Safety & Guardrails
- **Scope**: “Approved Toll documents only” for answers; block external browsing in MVP.
- **Prompt-injection & data-leak**: screen user input and retrieved text; strip tool-invocation attempts.
- **Operational/medical**: refuse SOPs or real-time guidance; provide official channels (e.g., 1800 776 902); offer consult booking.
- **Abuse & rate limiting**: throttle by IP/session; CAPTCHA for contact submissions.
- **DLP checks** (optional later): detect sensitive strings (keys, credentials) before sending emails or storing notes.

## Compliance posture (directional)
- Align to **least-privilege** and **encryption** controls typical of ISO 27001.
- Keep an audit trail suitable for incident response.
- Add privacy notice + consent text to the UI before lead capture.

## Incident handling (baseline)
- Classify: security vs privacy vs availability incident.
- Contain: revoke keys, pause tool, limit blast radius.
- Notify: internal stakeholders; follow legal/privacy notification rules as required.
