# End-to-End System Flow

## Diagram
```mermaid
flowchart LR
  user([User]) --> ui[Web Chat UI]
  ui --> api[API - FastAPI]
  api --> mcp[MCP Gateway]
  mcp --> orch[Orchestrator - LangGraph]

  %% Agents
  subgraph Agents
    router[Router Agent]
    retriever[Retriever Agent]
    verifier[Verifier Agent]
    intake[Intake Agent]
    scheduler[Scheduler Agent]
    emailer[Email Agent]
    crm[CRM Agent]
    guard[Guardrails Agent]
  end

  orch --> router
  router -->|ask_docs| retriever
  router -->|contact_*| intake
  router -->|ops_guarded| guard

  %% MCP Tools
  subgraph Tools_MCP[Model Context Protocol - tools]
    docsS[docs.search]
    kgE[kg.expand]
    verifyQ[verify.quote]
    calFind[calendar.find_slots]
    calCreate[calendar.create_event]
    emailSend[email.send]
    crmUpsert[crm.upsert_lead]
    validate[validate.contact]
  end

  retriever --> docsS
  retriever --> kgE
  verifier --> verifyQ
  intake --> validate
  intake --> calFind
  scheduler --> calCreate
  emailer --> emailSend
  crm --> crmUpsert

  %% Data Layer
  subgraph Data_Layer
    neo[(Neo4j - KG + vectors + fulltext)]
    redis[(Redis Cache)]
  end

  docsS --> neo
  kgE --> neo
  verifyQ --> neo

  calFind --> calendars[(Calendar API)]
  calCreate --> calendars
  emailSend --> mail[(Email API)]
  crmUpsert --> crms[(CRM)]

  %% Context and ingestion
  subgraph Context[Context - Toll documents]
    pages[Pages and Sections]
    chunks[Chunks with embeddings]
    meta[Metadata - title url dates]
  end

  ingest[Ingestion pipeline] -->|extract| pages
  pages -->|chunk| chunks
  chunks -->|embed| neo
  meta --> neo

  %% Answering and contact
  retriever --> verifier
  verifier --> ui

  intake --> scheduler
  scheduler --> emailer
  scheduler --> crm
  emailer --> ui
