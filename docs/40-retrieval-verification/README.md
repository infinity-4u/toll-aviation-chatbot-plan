# Retrieval & Verification

## Principles
- **Docs-only**: answers come strictly from approved Toll sources (About/History/Services).
- **Hybrid retrieval**: vector similarity **+** full-text **+** small KG boosts (all in Neo4j).
- **Sentence-level verification**: every sentence must be backed by a quote; otherwise it’s dropped or replaced with “I don’t know from the provided Toll documents.”
- **Clear citations**: inline markers + a “Sources” list (title + URL).
- **Safety routing**: operational/medical requests → guardrails (no SOPs or live guidance).

## Retrieval stack (Neo4j)
- **Vector indexes**: per-service (`uas`, `ace`, later `aero_fixed`, `defence`) **and** a global index.
- **Full-text index**: BM25-style keyword search over chunks.
- **KG boost**: use nearby entities (platforms, bases, programs, courses) to expand the query.

## Retrieval flow (MVP)
1. **Normalize + scope**: detect service (if any).
2. **Search**:  
   - Vector top **12** in the scoped index (fallback to global).  
   - Full-text top **12**.  
3. **Merge & expand (optional)**: expand with 1-hop KG terms; run a second pass if initial recall is weak.
4. **Rerank** to top **k=5** (cross-encoder or LLM reranker).

## Verification flow
- For each **candidate sentence** in the draft answer:
  - Find ≥1 **supporting quote** from retrieved chunks (include page title + URL).
  - Assign **confidence** (high/medium/low).  
  - **Drop** sentences with no support or **mark unknown**.
- If overall support is poor → respond with a concise **“I don’t know…”** message.

## Output format (to UI)
- Short bullets or concise paragraphs.  
- **Inline citations** (e.g., [1], [2]) and a **Sources** section listing title + URL.  
- Related topics (from KG neighbors) may appear in the right-rail.

## Supported query types (now)
- **About Toll Aviation** (mission, sectors, workforce).  
- **History / Timeline** (e.g., 2014 NSW/ACT contract, 2016 ACE opening, awards).  
- **Services** overview (UAS, ACE; later Fixed-Wing/Aeromedical, Defence).

## Diagram
```mermaid
graph TD
  Q[User Question] --> R[Hybrid Retrieve (vector + FT + KG)]
  R --> V[Verify sentence by sentence]
  V -->|unsupported| X[Drop / “I don’t know”]
  V -->|supported| G[Generate with citations]
  G --> A[Answer + Sources]

