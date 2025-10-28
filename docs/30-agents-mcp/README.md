# Agents & MCP Tools

## Minimal Agent Roster (MVP)

- **Router Agent** — Classify intent and service scope
- **Retriever Agent** — Run hybrid search in Neo4j (vector + full-text + KG)
- **Verifier Agent** — Sentence-level quote-back; drop unsupported claims
- **Contact Intake Agent** — Collect fields, validate, confirm consent
- **Scheduler Agent** — Propose 3 slots; on confirm, create event
- **Email Agent** — Send confirmations to user + team inbox
- **CRM Agent** — Upsert lead with reason + service
- **Guardrails Agent** — Screen for prompt injection / data leak / ops requests

## MCP Tool Contracts

### Documentation & Knowledge
- `docs.search(query, k, service?)`
- `kg.expand(entities, depth)`
- `verify.quote(sentences, chunks, threshold)`

### Contact & Scheduling
- `calendar.find_slots(participants, duration, window, tz)`
- `calendar.create_event(title, start, end, attendees, notes)`
- `email.send(to, cc, subject, html, thread_key)`
- `crm.upsert_lead(lead_json)`
- `validate.contact({email, phone})`

### Security
- `guardrails.screen(query)`

**Related:** [Intents & Routing](../10-intents-routing/README.md) • [Contact Flows](../50-contact-flows/README.md)
