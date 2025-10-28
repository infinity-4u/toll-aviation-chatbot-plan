# Agents & MCP Tools

## Minimal agent roster (MVP)
- **Router Agent** — classify intent and service scope.
- **Retriever Agent** — run hybrid search in Neo4j (vector + full-text + KG).
- **Verifier Agent** — sentence-level quote-back; drop unsupported claims.
- **Contact Intake Agent** — collect fields, validate, confirm consent.
- **Scheduler Agent** — propose 3 slots; on confirm, create event.
- **Email Agent** — send confirmations to user + team inbox.
- **CRM Agent** — upsert lead with reason + service.
- **Guardrails Agent** — screen for prompt injection / data leak / ops requests.

## MCP tool contracts (names only)
- `docs.search(query, k, service?)`
- `kg.expand(entities, depth)`
- `verify.quote(sentences, chunks, threshold)`
- `guardrails.screen(query)`
- `calendar.find_slots(participants, duration, window, tz)`
- `calendar.create_event(title, start, end, attendees, notes)`
- `email.send(to, cc, subject, html, thread_key)`
- `crm.upsert_lead(lead_json)`
- `validate.contact({email, phone})`

**Related:** [Intents & Routing](../10-intents-routing/README.md) • [Contact Flows](../50-contact-flows/README.md)
