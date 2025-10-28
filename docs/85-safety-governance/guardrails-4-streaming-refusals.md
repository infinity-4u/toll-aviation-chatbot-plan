# Streaming Refusals

**Goal**: Detect and block policy violations in real-time during streaming responses.

## Why This Matters

Traditional blocking happens AFTER full response generation. Streaming refusals enable:
- Stop harmful content mid-stream
- Faster user feedback
- Resource efficiency (stop expensive token generation early)

**Standard**: Real-time intervention with Claude 4 API.

## Implementation

### Claude 4 Streaming API Integration (Planned)

```python
import anthropic

client = anthropic.Anthropic(api_key="...")

with client.messages.stream(
    model="claude-4-sonnet",
    messages=[{"role": "user", "content": user_query}],
    max_tokens=1024
) as stream:
    for event in stream:
        if event.type == "content_block_delta":
            yield event.delta.text
        
        elif event.type == "message_stop":
            if event.message.stop_reason == "refusal":
                log.warning(f"Streaming refusal: {user_query}")
                yield "\n\n[Response blocked due to content policy. " \
                      "Contact 1800 776 902 for assistance.]"
```

**Trigger Scenarios**:
- Mid-stream policy violation detected
- PII accidentally being exposed
- Hallucination confidence drops below threshold
- Jailbreak pattern in model's reasoning

## Status

🚧 Planned for implementation after Claude 4 API GA release.