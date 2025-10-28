```md
# MCP Servers & Configuration (Toll Aviation MVP)

## Concrete server lineup

### 1) toll-docs (HTTP) — Docs Server
- **Tools:** `docs.search(query, scope, k)`, `verify.quote(text, chunk_ids)`
- **Resources:** `docs://pages/...`, `docs://chunks/...`
- **Reads:** Neo4j (chunks with embeddings, full-text, page metadata)

### 2) toll-kg (HTTP) — KG Server
- **Tools:** `kg.query(cypher)`, `kg.expand(entities, depth)`
- **Resources:** `kg://schema`, `kg://ontology`
- **Reads:** Neo4j

### 3) toll-contact (HTTP) — Contact Server
- **Tools:** `calendar.find_slots`, `calendar.create_event`, `email.send`, `crm.upsert_lead`
- **Resources/Prompts:** `/contact_intake` template, `/gdpr_notice`
- **Backends:** Calendar provider, email gateway, CRM

### 4) toll-guard (HTTP) — Guardrails Server
- **Tools:** `guardrails.screen(message)` for SOP/live-ops screening; returns decision + escalation text.

---

## Claude Code configuration (quick starts)

```bash
# Docs Server (HTTP)
claude mcp add --transport http toll-docs https://api.example.com/mcp/docs --scope project

# KG Server (HTTP)
claude mcp add --transport http toll-kg https://api.example.com/mcp/kg --scope project

# Contact Server (HTTP)
claude mcp add --transport http toll-contact https://api.example.com/mcp/contact --scope project

# Guardrails (HTTP)
claude mcp add --transport http toll-guard https://api.example.com/mcp/guard --scope project

# Verify registrations
claude mcp list
