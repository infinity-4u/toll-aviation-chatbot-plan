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

