# Roadmap & SLOs

## MVP Milestones

### Phase 1: Core Documentation Q&A (Weeks 1-4)
- Neo4j setup with vector + full-text indexes
- Ingest Toll Aviation content (About, History, Services)
- Basic retrieval + verification flow
- Simple chat UI with citations

### Phase 2: Contact Flows (Weeks 5-8)
- Contact form intake + validation
- Calendar integration (find slots, create events)
- Email confirmations
- CRM lead capture
- Triage routing (Training, Media, Tenders, Ops-guarded)

### Phase 3: Polish & Deploy (Weeks 9-12)
- Security hardening (secrets, TLS, RBAC)
- Observability (OpenTelemetry traces/metrics)
- Load testing + optimization
- Production deployment
- Internal beta testing

## Service Level Objectives (SLOs)

### Availability
- **Target**: 99.5% uptime (monthly)
- **Measurement**: API health checks every 60s

### Performance
- **Ask-Docs queries**: p95 ≤ 3s end-to-end
- **Booking flow**: p95 ≤ 6s (intake → confirmation)
- **Measurement**: OpenTelemetry spans

### Quality
- **Citation accuracy**: ≥95% (human eval on sample queries)
- **Booking completion rate**: ≥60% (users who start → complete)
- **Measurement**: Manual review + funnel analytics

### Safety
- **Zero operational/medical SOP leaks** in production
- **PII redaction**: 100% (automated + spot checks)

## Future Enhancements (Post-MVP)

- Multi-service expansion (Aeromedical, Defence, Fixed-Wing)
- Advanced KG reasoning (multi-hop queries)
- Voice interface integration
- Multi-tenant isolation (Neo4j Fabric)
- Real-time status updates for bookings
