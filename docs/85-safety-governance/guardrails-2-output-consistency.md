# Increase Output Consistency

**Goal**: Ensure structured, predictable responses for downstream system integration.

## Why This Matters

Inconsistent outputs break CRM integrations (malformed lead data), calendar bookings (missing fields), and email templates (broken HTML).

**Standard**: Typed schemas with 100% validation.

## Implementation

### Strict JSON Schemas (All MCP Tools)

Every tool response validated via Pydantic:

```python
from pydantic import BaseModel, Field
from typing import List, Literal

class Chunk(BaseModel):
    content: str = Field(..., min_length=10, max_length=5000)
    source: str = Field(..., pattern=r"^https?://.*")
    service: Literal["uas", "ace", "corporate"] 
    score: float = Field(..., ge=0.0, le=1.0)
    metadata: dict

class SearchResult(BaseModel):
    chunks: List[Chunk] = Field(..., min_items=0, max_items=20)
    total_results: int = Field(..., ge=0)
    query_time_ms: float = Field(..., gt=0)
    
    @validator('chunks')
    def chunks_sorted_by_score(cls, v):
        scores = [chunk.score for chunk in v]
        if scores != sorted(scores, reverse=True):
            raise ValueError("Chunks must be sorted by score descending")
        return v
```

### Response Prefilling for Contact Workflows

Forces structured output without preambles:

```python
# Assistant prefill
"""
<booking_intake>
  <name>John Smith</name>
  <email>Not provided yet</email>
  <phone>Not provided yet</phone>
  <inquiry>UAS training inquiry</inquiry>
  <status>Will ask</status>
  <next_action>Ask for email and phone</next_action>
</booking_intake>
"""
```

**Benefit**: 100% parseable output, no freeform text.