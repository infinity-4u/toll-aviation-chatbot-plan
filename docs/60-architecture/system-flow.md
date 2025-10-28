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


flowchart LR
  U([User]) --> H[Claude Code Host]

  %% MCP clients to servers
  H --> S1[Docs Server]
  H --> S2[KG Server]
  H --> S3[Contact Server]
  H --> S4[Guardrails Server]

  %% Data plane
  subgraph DATA[Data Layer]
    N[(Neo4j 5\nKG + vectors + fulltext)]
    R[(Redis Cache)]
  end

  %% Ingestion
  subgraph INGEST[Ingestion]
    P[HTML pages]
    C[Chunks + embeddings]
    M[Metadata]
  end

  P --> C --> N
  M --> N

  %% Servers to data/external
  S1 --> N
  S2 --> N

  subgraph EXTERNAL[External Services]
    CAL[(Calendar API)]
    MAIL[(Email API)]
    CRM[(CRM API)]
  end

  S3 --> CAL
  S3 --> MAIL
  S3 --> CRM

  %% Answers and contact flows
  S1 --> H
  S2 --> H
  S4 --> H
  S3 --> H
  H --> U

