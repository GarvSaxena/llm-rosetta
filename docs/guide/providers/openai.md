# OpenAI

OpenAI is supported through two shim endpoints covering both the Chat Completions API and the newer Responses API.

## Endpoints

| Shim Name | Base Type | Default Base URL | API Key Env |
|:---|:---|:---|:---|
| `openai--chat_completions` | `chat_completions` | `https://api.openai.com/v1` | `OPENAI_API_KEY` |
| `openai--open_responses` | `open_responses` | `https://api.openai.com/v1` | `OPENAI_API_KEY` |

## Reasoning Support

Both endpoints support reasoning/thinking with slightly different field names matching their respective API standards.

**Chat Completions (`openai--chat_completions`):**

| Field | Value |
|:---|:---|
| Effort field | `reasoning_effort` |
| Effort range | `minimal` → `high` |
| Visibility modes | `auto`, `concise`, `detailed` |

**Responses API (`openai--open_responses`):**

| Field | Value |
|:---|:---|
| Effort field | `reasoning.effort` |
| Effort range | `minimal` → `high` |
| Visibility modes | `auto`, `concise`, `detailed` |

## Capability Flags

| Flag | `openai--chat_completions` | `openai--open_responses` |
|:---|:---:|:---:|
| Custom tools | ✅ | ✅ |
| Max tool description | 1024 chars | — |
| Native tool search | — | ✅ |

## Transforms

Both endpoints apply `hoist_late_system_messages()` to consolidate any late system messages to the beginning of the conversation. No fields are stripped or renamed.

## Gateway Configuration

=== "Chat Completions"

    ```jsonc
    {
      "providers": {
        "openai": {
          "shim": "openai--chat_completions",
          "api_key": "${OPENAI_API_KEY}"
        }
      }
    }
    ```

=== "Responses API"

    ```jsonc
    {
      "providers": {
        "openai-responses": {
          "shim": "openai--open_responses",
          "api_key": "${OPENAI_API_KEY}"
        }
      }
    }
    ```

## Notes

- The `openai--chat_completions` shim uses the Chat Completions format (`/v1/chat/completions`), while `openai--open_responses` uses the Responses format (`/v1/responses`). Choose based on which downstream client format you need.
- Response ID prefixes differ: `chatcmpl-` for Chat Completions, `resp_` for Responses API.
