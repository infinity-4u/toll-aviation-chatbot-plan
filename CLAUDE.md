# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a **planning repository** for the Toll Aviation Chatbot project - a documentation-first approach to building a trustworthy AI chatbot with knowledge retrieval and contact capabilities. The repository contains structured documentation, architectural decisions, and planning materials rather than implementation code.

## Documentation Structure

The repository follows a numbered documentation structure organized in `docs/`:

- `00-overview/` - High-level plan, scope, and success criteria
- `10-intents-routing/` - User request classification and routing
- `20-kg-schema/` - Neo4j knowledge graph structure
- `30-agents-mcp/` - MCP tool contracts and agent definitions  
- `40-retrieval-verification/` - Hybrid search and citation verification
- `50-contact-flows/` - Booking, email, and CRM workflows
- `60-architecture/` - System design (simplified MCP pattern)
- `70-tech-stack-infra/` - Technology choices and deployment
- `80-security-privacy/` - PII handling, compliance, guardrails
- `90-roadmap-slo/` - Milestones and service level objectives

## Key Commands

### Documentation Development
```bash
# Install MkDocs (if not available)
pip install mkdocs mkdocs-material pymdown-extensions

# Serve documentation locally
mkdocs serve

# Build static documentation
mkdocs build

# Deploy to GitHub Pages (automatic via GitHub Actions)
mkdocs gh-deploy
```

### Documentation Site
- **Local**: http://127.0.0.1:8000 (when running `mkdocs serve`)
- **Live**: https://infinity-4u.github.io/toll-aviation-chatbot-plan/

## Architecture Overview

**Design Pattern**: Simplified MCP (Model Context Protocol) architecture
- **Claude Host** orchestrates (no separate LangGraph orchestrator)
- **Stateless MCP servers** for horizontal scaling
- **Gateway pattern** for auth, rate limiting, request tracking
- **Observable by default** with OpenTelemetry

### Core Components
1. **Frontend**: React + TypeScript web UI
2. **API Layer**: FastAPI + MCP Gateway + Redis cache
3. **MCP Servers**: Docs, Contact, KG, Guard (all stateless)
4. **Data Layer**: Neo4j 5 (vectors + full-text + knowledge graph)
5. **Observability**: OpenTelemetry → Prometheus + Grafana + Loki

### MCP Tool Contracts
- **Documentation**: `docs.search()`, `kg.expand()`, `verify.quote()`
- **Contact**: `calendar.find_slots()`, `email.send()`, `crm.upsert_lead()`
- **Security**: `guardrails.screen()`

## Project Goals

Building a **dual-purpose chatbot**:
1. **Ask-Docs**: Answer questions about Toll Aviation with verified citations (≥95% accuracy)
2. **Contact Actions**: Complete booking/inquiry workflows end-to-end (≤6s p95 latency, ≥60% completion rate)

## Key Constraints

- **Docs-only knowledge** - No external web browsing
- **Operational guardrails** - Refuses SOP/medical/flight guidance
- **PII protection** - Data minimization, consent, 30-day retention
- **Citation requirement** - Every sentence must be backed by source

## Documentation Workflow

1. **Edit**: Modify markdown files in `docs/`
2. **Test**: Run `mkdocs serve` to preview locally
3. **Commit**: Push to `main` branch
4. **Deploy**: GitHub Actions automatically deploys to GitHub Pages

## Related Files

- `mkdocs.yml` - MkDocs configuration
- `docs/index.md` - Main documentation homepage
- `README.md` - Repository overview
- `CHANGELOG.md` - Version history

## Working with This Repository

This is a **living plan** that evolves during development. When working on implementation:

1. **Reference architecture** in `docs/60-architecture/architecture-v2.md`
2. **Check tool contracts** in `docs/30-agents-mcp/README.md` 
3. **Review security requirements** in `docs/80-security-privacy/README.md`
4. **Update documentation** as decisions change
5. **Maintain architectural decision record** via documentation updates

The documentation serves as the single source of truth for technical decisions and system design.