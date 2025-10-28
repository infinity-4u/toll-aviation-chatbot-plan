# Knowledge Strategy & Schema (One KG)

**Choice**: Single Neo4j database / ontology with **service-scoped subgraphs**. Fabric-ready to split later.

**Backbone**: Org, Service, Program/Contract, Capability, Platform/AssetType, Fleet, Base/Facility→Place, Course, Simulator, Certification, ClientOrg, Event, Award, KPI, Page/Section/Chunk{embedding}, Source.

**Timeline**: (:Org)-[:ACHIEVED]->(:Event {label, year_start, year_end, notes})

**Split triggers**: legal isolation; cross-links <5% for 90 days; p95 breaches.
