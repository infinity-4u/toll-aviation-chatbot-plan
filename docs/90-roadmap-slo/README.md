```md
# Roadmap & SLOs

## Phases (high level)
1) **Foundation**  
   - Ingest About/History + UAS/ACE pages into Neo4j (chunks + metadata).  
   - Hybrid retrieval + citations.  
2) **Contact flows**  
   - Intake, triage, calendar slots, event creation, emails, CRM lead.  
3) **KG enrichment**  
   - Courses, simulators, platforms, bases; Related Topics UI.  
4) **Hardening**  
   - SSO, guardrails, reCAPTCHA, observability dashboards; Fabric-ready.

## Service Levels (targets)
- **Ask-Docs latency**: p95 ≤ 3s  
- **Booking end-to-end**: p95 ≤ 6s  
- **Citation accuracy**: ≥ 95%  
- **Triage precision**: ≥ 90%  
- **Booking completion (initiated)**: ≥ 60%

## Quality checks
- Sentence-level verification required for all claims.  
- Refusal policy for operational/medical requests.  
- Privacy consent must be confirmed before CRM upsert.

## Release tagging
- Tag plan snapshots as `plan-vX.Y.Z`.  
- Update `CHANGELOG.md` for each tag.  
- Add/adjust ADRs when architectural decisions change.
