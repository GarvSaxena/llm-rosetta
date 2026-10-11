# MiniMax

MiniMax is supported through two shim endpoints covering both OpenAI Chat Completions and Anthropic Messages API formats.

## Endpoints

| Shim Name | Base Type | Default Base URL | API Key Env |
|:---|:---|:---|:---|
| `minimax--chat_completions` | `chat_completions` | `https://api.minimaxi.com/v1` | `MINIMAX_API_KEY` |
| `minimax--messages` | `messages` | `https://api.minimaxi.com/anthropic` | `MINIMAX_API_KEY` |

## Reasoning Support

**Chat Completions (`minimax--chat_completions`):**

| Field | Value |
|:---|:---|
| Effort range | `low` → `high` |
| Thinking modes | `auto` → `adaptive`, `enabled`, `disabled` |

**Anthropic (`minimax--messages`):**

| Field | Value |
|:---|:---|
| Effort field | `output_config.effort` |
| Effort range | `low` → `max` |
| Thinking modes | `auto` → `adaptive`, `enabled`, `disabled` |

## Transforms

| Shim | Type | Transform | Purpose |
|:---|:---|:---|:---|
| `minimax--chat_completions` | Pre-IR | `parse_think_tags()` | Parses `<think>...</think>` tags in responses into structured `reasoning_content` |
| `minimax--chat_completions` | Post-IR | `strip_fields("logprobs", "top_logprobs", "seed", "stop")` | Removes unsupported request fields |
| `minimax--chat_completions` | Post-IR | `inject_reasoning_split()` | Sets `reasoning_split: true` when thinking is present in the request |
| `minimax--messages` | IR | `auto_cache_breakpoints()` | Inserts cache breakpoints for prompt caching |
| Both | IR | `hoist_late_system_messages()` | Moves late system messages to conversation start |

## Gateway Configuration

=== "Chat Completions"

    ```jsonc
    {
      "providers": {
        "minimax": {
          "shim": "minimax--chat_completions",
          "api_key": "${MINIMAX_API_KEY}"
        }
      }
    }
    ```

=== "Anthropic"

    ```jsonc
    {
      "providers": {
        "minimax-anthropic": {
          "shim": "minimax--messages",
          "api_key": "${MINIMAX_API_KEY}"
        }
      }
    }
    ```

## Notes

- The Chat Completions endpoint includes special handling for MiniMax's `<think>` tag format. When the model returns reasoning in `<think>...</think>` tags instead of structured `reasoning_content`, the shim automatically parses these into the standard reasoning format.
- The `reasoning_split` flag is automatically injected into requests when thinking is enabled, signaling MiniMax to separate reasoning from the final response.
