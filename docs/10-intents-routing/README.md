# Intents & Routing

**Intents**: ask_docs | contact_book | contact_call | contact_email | ops_guarded  
**Service scope**: uas | ace | aero_fixed | defence | unknown

**How this maps to MCP tools**  
- ask_docs → `docs.search` (+ optional `kg.expand`) → `verify.quote`  
- contact_book / contact_call → `calendar.find_slots` → `calendar.create_event` → `email.send` → `crm.upsert_lead`  
- contact_email → `email.send` → `crm.upsert_lead`  
- ops_guarded → `guardrails.screen` + provide official channels

**See also:** [Agents & MCP Tools](../30-agents-mcp/README.md) • [Contact Flows](../50-contact-flows/README.md)

## General Contact triage
Training → ACE/UAS; Operational → ops_guarded; Media → media inbox; Tender → BD/Tenders; Other → manual.

```mermaid
graph TD
  UQ[User Input] --> RT[Router]
  RT -->|ask_docs| RET[Retriever]
  RT -->|contact_*| CT[Intake]
  RT -->|ops_guarded| GD[Guardrails]
  RET --> VF[Verifier] --> OUT[Answer + citations]
  CT --> SCH[Scheduler] --> EVT[Create Event] --> EM[Email] --> CRM[Lead]
  GD --> SAFE[Refusal + official channels]
