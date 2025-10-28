# Knowledge Strategy & Schema (One KG)

## Strategy Choice

Single Neo4j database / ontology with **service-scoped subgraphs**. Fabric-ready to split later.

## Backbone Entities

- **Organization:** Org, Service, Program/Contract, Capability, ClientOrg
- **Operations:** Platform/AssetType, Fleet, Base/Facility→Place
- **Training:** Course, Simulator, Certification
- **Content:** Page/Section/Chunk{embedding}, Source
- **Tracking:** Event, Award, KPI

## Timeline Model
```cypher
