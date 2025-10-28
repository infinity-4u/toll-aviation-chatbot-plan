# Architecture

## Components
- **Frontend**: React + TypeScript (chat UI, citations, forms)
- **API**: FastAPI
- **Orchestrator**: LangGraph (agents)
- **Datastore**: Neo4j 5 (vectors + full-text + KG), Redis (cache)
- **Integrations via MCP**: Calendar, Email, CRM
- **Observability**: OpenTelemetry (metrics/logs/traces)

## Diagram
```mermaid
flowchart LR
  UI[Web UI: React + TS] --> API[FastAPI API]
  API --> MCP[MCP Gateway]
  MCP --> ORCH[Agent Orchestrator: LangGraph]
  ORCH --> NEO[(Neo4j 5: Vectors + Full-text + KG)]
  ORCH --> CAL[Calendar API]
  ORCH --> MAIL[Email API]
  ORCH --> CRM[(CRM API)]
  API --> REDIS[(Redis Cache)]
