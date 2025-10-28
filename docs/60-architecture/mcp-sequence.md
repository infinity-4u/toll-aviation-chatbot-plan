sequenceDiagram
  participant U as User
  participant H as Claude Host
  participant D as Docs Server
  participant K as KG Server
  participant C as Contact Server
  participant N as Neo4j

  U->>H: Message
  H->>D: initialize (negotiate capabilities)
  D-->>H: ready (tools/resources/prompts)
  H->>D: tools/list
  D-->>H: tool metadata
  H->>D: tools/call docs.search
  D->>N: query (vector + fulltext + filters)
  N-->>D: chunks
  D-->>H: content
  H->>D: tools/call verify.quote
  D->>N: find quotes
  D-->>H: citations
  H-->>U: Answer and Sources

  rect rgb(245,245,245)
  Note over H: if intent = contact_*
  H->>C: tools/call calendar.find_slots
  C-->>H: slots
  H-->>U: propose slots
  U-->>H: confirm
  H->>C: calendar.create_event / email.send / crm.upsert
  C-->>H: confirmation
  H-->>U: confirmation
  end
