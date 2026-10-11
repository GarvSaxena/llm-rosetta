---
title: Auto Detection
---

# Auto Detection

LLM-Rosetta can automatically detect which provider format a request uses.

## Detecting Provider

```python
from llm_rosetta import detect_provider

provider = detect_provider(request_dict)
# Returns: "chat_completions", "open_responses", "messages", "google_generate", "google_interactions", or None
```

## Getting a Converter

```python
from llm_rosetta import get_converter_for_provider

converter = get_converter_for_provider("messages")
```

## Convenience Conversion

```python
from llm_rosetta import convert

# Auto-detect source, convert to target
result = convert(
    source_body=openai_request,
    target_provider="messages",
    source_provider=None,  # auto-detect
)
```

### Force Conversion for Same-Provider Requests

By default, `convert()` returns the body as-is when the source and target providers are the same. Use `force_conversion=True` to run the full conversion pipeline even in this case — useful for parameter normalization:

```python
# Normalize OpenAI Chat parameters (e.g. max_tokens → max_completion_tokens)
result = convert(
    source_body=openai_request,
    target_provider="chat_completions",
    force_conversion=True,
)
```
