# Tech Stack & Infrastructure

## Application stack
- **Frontend**: React + TypeScript, Tailwind CSS, React Query
- **Backend**: Python 3.11, FastAPI, LangGraph (agents), MCP Python SDK
- **Datastore**: Neo4j 5 (vectors + full-text + KG in one), Redis (cache)
- **Optional analytics**: Postgres (lead/events journaling)

## Integrations (via MCP tools)
- **Calendar**: O365 or Google Calendar
- **Email**: O365 or Gmail
- **CRM**: Salesforce or HubSpot

## Environments
- **Dev**: single-node Neo4j, minimal keys
- **Staging**: Neo4j Aura or small cluster; test calendars/emails/CRM sandboxes
- **Prod**: Neo4j Aura/cluster; separate service accounts and secrets

## CI/CD
- **GitHub Actions**: lint/links check on PR; build artifacts (docs site optional)
- **Docker** images for backend services
- **Kubernetes** (later): horizontal autoscaling on p95 latency

## Observability
- **OpenTelemetry** for traces/metrics/logs
- **Dashboards**: Grafana/Loki/Tempo (or vendor equivalent)

## Secrets & config
- Store in cloud **Secrets Manager/KMS**
- Never place PII or secrets in prompts or logs
