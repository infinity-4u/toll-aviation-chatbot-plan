# Security, Privacy, Safety

## Privacy & PII

- **Data minimisation**: Collect only what the form needs (First/Last, Email, Phone, Company, Reason, Message, Consent)
- **Consent**: Explicit privacy + contact consent before creating a lead
- **Retention**: Default 30 days for raw chat + contact payloads unless extended by policy
- **Redaction**: Remove PII from logs/telemetry; never include PII in prompts

## Security

- **Transport & storage**: TLS in transit; encrypted-at-rest for all stores (Neo4j, Redis, Postgres if used)
- **Secrets**: Store in cloud Secrets Manager/KMS; rotate; never commit to Git
- **Access control**: Least-privilege service accounts for Calendar/Email/CRM; RBAC on infra; audited actions
- **Auditability**: Structured logs for tool calls (who/what/when), without PII content

## Safety & Guardrails

- **Scope**: "Approved Toll documents only" for answers; block external browsing in MVP
- **Prompt-injection & data-leak**: Screen user input and retrieved text; strip tool-invocation attempts
- **Operational/medical**: Refuse SOPs or real-time guidance; provide official channels (e.g., 1800 776 902); offer consult booking
- **Abuse & rate limiting**: Throttle by IP/session; CAPTCHA for contact submissions
- **DLP checks** (optional later): Detect sensitive strings (keys, credentials) before sending emails or storing notes

## Compliance Posture (Directional)

- Align to **least-privilege** and **encryption** controls typical of ISO 27001
- Keep an audit trail suitable for incident response
- Add privacy notice + consent text to the UI before lead capture

## Incident Handling (Baseline)

- **Classify**: Security vs privacy vs availability incident
- **Contain**: Revoke keys, pause tool, limit blast radius
- **Notify**: Internal stakeholders; follow legal/privacy notification rules as required
