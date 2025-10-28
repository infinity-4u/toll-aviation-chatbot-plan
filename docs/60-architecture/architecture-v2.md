# Architecture v2 (Simplified MCP)

## Design Principles

- **Claude Host orchestrates** - No separate LangGraph orchestrator needed for MVP
- **Stateless MCP servers** - Each server is independently scalable
- **Gateway pattern** - Single entry point for auth, rate limiting, request tracking
- **Observable by default** - OpenTelemetry spans on every tool call
- **PII-safe logging** - Automatic redaction in all log outputs

---

## High-Level Architecture
```mermaid
flowchart TB
    subgraph CLIENT[Client Layer]
        UI[Web UI: React + TypeScript]
        MOB[Mobile App: Future]
    end
    
    subgraph API[API Layer]
        FE[FastAPI Frontend]
        GW[MCP Gateway]
        REDIS[(Redis Cache)]
    end
    
    subgraph MCP[MCP Server Layer]
        DS[Docs Serversearch, verify]
        KGS[KG Serverexpand, query]
        CS[Contact Servercalendar, email, crm]
        GS[Guard Serverscreen, validate]
    end
    
    subgraph DATA[Data Layer]
        NEO[(Neo4j 5vectors + FT + KG)]
    end
    
    subgraph EXT[External Services]
        CAL[Calendar API]
        MAIL[Email API]
        CRM[CRM API]
    end
    
    subgraph OBS[Observability]
        OTEL[OpenTelemetryCollector]
        PROM[Prometheus]
        GRAF[Grafana]
        LOGS[Loki]
    end
    
    UI --> FE
    MOB -.-> FE
    FE --> GW
    FE --> REDIS
    
    GW --> DS
    GW --> KGS
    GW --> CS
    GW --> GS
    
    DS --> NEO
    KGS --> NEO
    
    CS --> CAL
    CS --> MAIL
    CS --> CRM
    
    DS --> OTEL
    KGS --> OTEL
    CS --> OTEL
    GS --> OTEL
    GW --> OTEL
    
    OTEL --> PROM
    OTEL --> LOGS
    PROM --> GRAF
    LOGS --> GRAF
```

---

## Component Details

### 1. Frontend (React + TypeScript)
- Chat interface with streaming responses
- Citation display with clickable sources
- Contact forms with validation
- Booking calendar widget
- Mobile-responsive design

### 2. FastAPI Frontend API
- Session management
- WebSocket for streaming
- Static asset serving
- Rate limiting per user/IP
- CORS configuration

### 3. MCP Gateway
**Responsibilities:**
- Request ID generation and propagation
- Authentication and authorization
- Rate limiting (by client, by tool)
- Request/response logging (PII-redacted)
- Health check aggregation
- Circuit breaking for external services

**Not included in MVP:**
- Complex orchestration (Claude Host does this)
- Business logic (lives in MCP servers)

### 4. MCP Servers (Stateless)

#### Docs Server
**Tools:**
- `docs.search(query, k, service?)`
- `verify.quote(sentences, chunks, threshold)`

**Features:**
- Hybrid search (vector + full-text + KG boost)
- Sentence-level verification
- Citation generation
- PII detection and redaction

#### KG Server
**Tools:**
- `kg.query(cypher)`
- `kg.expand(entities, depth)`

**Features:**
- Ontology navigation
- Entity relationship exploration
- Timeline queries
- Service-scoped subgraph queries

#### Contact Server
**Tools:**
- `calendar.find_slots(participants, duration, window, tz)`
- `calendar.create_event(title, start, end, attendees, notes)`
- `email.send(to, cc, subject, html, thread_key)`
- `crm.upsert_lead(lead_json)`
- `validate.contact({email, phone})`

**Features:**
- Multi-calendar coordination
- Email template rendering
- CRM field mapping
- GDPR consent handling

#### Guard Server
**Tools:**
- `guardrails.screen(message)`

**Features:**
- Prompt injection detection
- Data leak prevention
- Operational/medical request screening
- Escalation routing

### 5. Data Layer

#### Neo4j 5
- **Vector indexes**: Per-service (uas, ace) + global
- **Full-text indexes**: BM25 over chunks
- **Knowledge graph**: Org, Service, Program, Capability, Platform, Course, etc.
- **Backup**: Daily automated snapshots

#### Redis
- Session cache (30-min TTL)
- API response cache (configurable per endpoint)
- Rate limit counters
- Distributed locks for booking

### 6. Observability Stack

#### OpenTelemetry
- **Traces**: End-to-end request tracking
- **Metrics**: Tool call counts, latencies, errors
- **Logs**: Structured JSON with request IDs

#### Prometheus + Grafana
- SLO dashboards (p95 latency, error rates)
- Capacity planning metrics
- Alert rules for SLO breaches

#### Loki
- Centralized log aggregation
- PII-redacted audit logs
- Retention: 90 days hot, 1 year cold

---

## Detailed Request Flow

### Ask-Docs Flow
```mermaid
sequenceDiagram
    participant U as User
    participant UI as Web UI
    participant API as FastAPI
    participant GW as MCP Gateway
    participant CH as Claude Host
    participant DS as Docs Server
    participant NEO as Neo4j
    
    U->>UI: Ask question
    UI->>API: POST /chat
    API->>GW: Forward with Request-ID
    
    GW->>CH: Initialize conversation
    
    rect rgb(240, 248, 255)
        Note over CH,DS: Tool Discovery Phase
        CH->>DS: GET /tools/list
        DS-->>CH: [docs.search, verify.quote]
    end
    
    rect rgb(255, 250, 240)
        Note over CH,NEO: Search Phase
        CH->>GW: Call docs.search
        GW->>DS: docs.search(query, k=12, service="uas")
        DS->>NEO: Vector + FT search
        NEO-->>DS: Top 12 chunks
        DS->>NEO: Optional KG expansion
        NEO-->>DS: Related entities
        DS-->>GW: Merged results
        GW-->>CH: chunks[0..11]
    end
    
    rect rgb(240, 255, 240)
        Note over CH,NEO: Verification Phase
        CH->>GW: Call verify.quote
        GW->>DS: verify.quote(sentences, chunks)
        DS->>NEO: Find supporting quotes
        NEO-->>DS: Quote matches
        DS-->>GW: citations + confidence
        GW-->>CH: verified answer
    end
    
    CH->>GW: Final response
    GW->>API: Response + telemetry
    API->>UI: Stream answer + sources
    UI->>U: Display with citations
    
    Note over GW,DS: All spans exported to OpenTelemetry
```

### Contact Flow
```mermaid
sequenceDiagram
    participant U as User
    participant UI as Web UI
    participant API as FastAPI
    participant GW as MCP Gateway
    participant CH as Claude Host
    participant CS as Contact Server
    participant CAL as Calendar API
    participant MAIL as Email API
    participant CRM as CRM API
    
    U->>UI: "Book a meeting about UAS training"
    UI->>API: POST /chat
    API->>GW: Forward with Request-ID
    GW->>CH: Process intent
    
    rect rgb(255, 240, 240)
        Note over CH,CS: Intake Phase
        CH->>U: Collect details (name, email, phone, reason)
        U->>CH: Provide information
        CH->>GW: validate.contact
        GW->>CS: validate.contact({email, phone})
        CS-->>GW: ✓ valid
        GW-->>CH: validation result
    end
    
    rect rgb(240, 248, 255)
        Note over CH,CAL: Scheduling Phase
        CH->>GW: calendar.find_slots
        GW->>CS: find_slots(duration=30min, window=7days, tz="Australia/Melbourne")
        CS->>CAL: Check availability
        CAL-->>CS: Available slots
        CS-->>GW: 3 proposed times
        GW-->>CH: slot options
        CH->>U: Propose 3 time slots
        U->>CH: Select slot #2
    end
    
    rect rgb(240, 255, 240)
        Note over CH,CRM: Booking Phase
        par Create Event
            CH->>GW: calendar.create_event
            GW->>CS: create_event(...)
            CS->>CAL: Create calendar event
            CAL-->>CS: Event ID
        and Send Confirmations
            CS->>MAIL: Email to user
            CS->>MAIL: Email to team inbox
            MAIL-->>CS: Message IDs
        and Update CRM
            CS->>CRM: upsert_lead({...})
            CRM-->>CS: Lead ID
        end
        
        CS-->>GW: All confirmations
        GW-->>CH: Success
        CH->>U: "✓ Booked for [date/time]"
    end
    
    Note over GW,CS: Telemetry exported at each step
```

---

## Request Tracking & Observability

### Request ID Propagation
```mermaid
flowchart LR
    UI[Web UI] -->|X-Request-ID: uuid| API[FastAPI]
    API -->|Same ID| GW[MCP Gateway]
    GW -->|Same ID| DS[Docs Server]
    GW -->|Same ID| CS[Contact Server]
    
    DS -->|Trace: request_id| OTEL[OpenTelemetry]
    CS -->|Trace: request_id| OTEL
    GW -->|Trace: request_id| OTEL
    
    OTEL -->|Export| PROM[Prometheus]
    OTEL -->|Export| LOGS[Loki]
```

### Instrumentation Points

Every MCP tool call automatically captures:

1. **Request metadata**
   - Request ID
   - Tool name
   - Parameters (PII-redacted)
   - Calling service

2. **Timing data**
   - Start time
   - Duration
   - Queue time (if applicable)

3. **Result metadata**
   - Success/failure
   - Result size
   - Cache hit/miss
   - Error type (if failed)

4. **Business metrics**
   - Search result count
   - Verification confidence
   - Booking completion rate
   - Escalation triggers

---

## Deployment Architecture

### Development
```
┌─────────────────┐
│  Local Machine  │
│  - React dev    │
│  - FastAPI      │
│  - Neo4j Docker │
│  - Redis Docker │
└─────────────────┘
```

### Staging
```
┌──────────────────────────┐
│    Azure/AWS/GCP         │
│  ┌────────────────────┐  │
│  │  App Service       │  │
│  │  - Frontend + API  │  │
│  └────────────────────┘  │
│  ┌────────────────────┐  │
│  │  Container Instances│ │
│  │  - MCP Servers (4) │  │
│  └────────────────────┘  │
│  ┌────────────────────┐  │
│  │  Managed Services  │  │
│  │  - Neo4j Aura      │  │
│  │  - Redis Cache     │  │
│  └────────────────────┘  │
└──────────────────────────┘
```

### Production
```
┌────────────────────────────────┐
│    Kubernetes Cluster          │
│  ┌──────────────────────────┐  │
│  │  Frontend Pods (3x)      │  │
│  │  - React + FastAPI       │  │
│  │  - Auto-scaling          │  │
│  └──────────────────────────┘  │
│  ┌──────────────────────────┐  │
│  │  MCP Server Pods         │  │
│  │  - Docs (3x)             │  │
│  │  - Contact (2x)          │  │
│  │  - KG (2x)               │  │
│  │  - Guard (2x)            │  │
│  └──────────────────────────┘  │
│  ┌──────────────────────────┐  │
│  │  Stateful Sets           │  │
│  │  - Neo4j cluster (3x)    │  │
│  │  - Redis cluster (3x)    │  │
│  └──────────────────────────┘  │
│  ┌──────────────────────────┐  │
│  │  Observability           │  │
│  │  - Prometheus            │  │
│  │  - Grafana               │  │
│  │  - Loki                  │  │
│  │  - Jaeger (traces)       │  │
│  └──────────────────────────┘  │
└────────────────────────────────┘
```

---

## Key Architectural Decisions

### ✅ What Changed from v1

| Aspect | v1 (Original) | v2 (Simplified) |
|--------|---------------|-----------------|
| Orchestration | LangGraph agent | Claude Host native |
| MCP Access | Direct from orchestrator | Through Gateway |
| Request Tracking | Manual | Automatic (Request ID) |
| Observability | Planned | Built-in (OpenTelemetry) |
| State Management | In orchestrator | Stateless servers |
| Scalability | Vertical (orchestrator) | Horizontal (servers) |

### 🎯 Why These Changes

1. **Removed LangGraph** - Claude Host already orchestrates tool calls intelligently. Adding LangGraph adds complexity without value for MVP.

2. **Added MCP Gateway** - Centralized auth, rate limiting, and request tracking. Easier to secure and monitor.

3. **Stateless Servers** - Each MCP server can scale independently based on load. No shared state = no bottlenecks.

4. **Observability First** - Built-in tracing means debugging production issues is straightforward.

5. **PII Safety** - Automatic redaction at gateway level protects customer data.

---

## Performance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Ask-Docs p95 | ≤ 3s | OpenTelemetry span |
| Booking E2E p95 | ≤ 6s | Multi-span trace |
| Availability | 99.5% | Health checks |
| Cache Hit Rate | ≥ 40% | Redis metrics |
| Error Rate | < 1% | Tool call success rate |

---

## Security Layers
```mermaid
flowchart TD
    A[User Request] --> B{Rate Limit}
    B -->|Pass| C{Authentication}
    B -->|Fail| X1[429 Too Many Requests]
    C -->|Valid| D{PII Detection}
    C -->|Invalid| X2[401 Unauthorized]
    D -->|Clean| E[MCP Gateway]
    D -->|Contains PII| F[Redact & Log]
    F --> E
    E --> G{Tool Call}
    G -->|Allowed| H[Execute]
    G -->|Blocked| X3[403 Forbidden]
    H --> I{Result Validation}
    I -->|Valid| J[Return]
    I -->|Invalid| X4[500 Internal Error]
```

---

## Migration Path (v1 → v2)

### Phase 1: Add Gateway (Week 1)
- Deploy MCP Gateway in front of existing servers
- Add request ID propagation
- No breaking changes

### Phase 2: Remove LangGraph (Week 2)
- Update API to call Claude Host directly
- Migrate state management to Redis
- Test all workflows

### Phase 3: Add Observability (Week 3)
- Deploy OpenTelemetry collectors
- Add instrumentation to servers
- Create Grafana dashboards

### Phase 4: Optimize (Week 4)
- Add Redis caching
- Tune Neo4j indexes
- Load testing and optimization

---

## Related Documentation

- [MCP Sequence Diagrams](./mcp-sequence.md) - Original sequence flows
- [System Flow](./system-flow.md) - Original architecture
- [Tech Stack](../70-tech-stack-infra/README.md) - Technology choices
- [Security & Privacy](../80-security-privacy/README.md) - Security controls
