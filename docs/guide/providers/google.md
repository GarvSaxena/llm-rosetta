# Google

Google is supported through two shim endpoints: one for the generateContent API and one for the newer Interactions API.

## Endpoints

| Shim Name | Base Type | Default Base URL | API Key Env |
|:---|:---|:---|:---|
| `google--google_generate` | `google_generate` | `https://generativelanguage.googleapis.com` | `GOOGLE_API_KEY` |
| `google--google_interactions` | `google_interactions` | `https://generativelanguage.googleapis.com` | `GOOGLE_API_KEY` |

## Reasoning Support

Both endpoints share the same reasoning configuration:

| Field | Value |
|:---|:---|
| Effort field | `thinking_level` |
| Effort range | `minimal` → `high` |

## Transforms

| Shim | Type | Transform | Purpose |
|:---|:---|:---|:---|
| `google--google_generate` | IR | `hoist_late_system_messages()` | Moves late system messages to conversation start |
| `google--google_interactions` | — | — | No transforms applied |

## Gateway Configuration

=== "generateContent"

    ```jsonc
    {
      "providers": {
        "google": {
          "shim": "google--google_generate",
          "api_key": "${GOOGLE_API_KEY}"
        }
      }
    }
    ```

=== "Interactions"

    ```jsonc
    {
      "providers": {
        "google-interactions": {
          "shim": "google--google_interactions",
          "api_key": "${GOOGLE_API_KEY}"
        }
      }
    }
    ```

## Notes

- The `google--google_generate` shim targets the generateContent endpoint, which is the standard Gemini API. The `google--google_interactions` shim targets the newer Interactions API format.
- Google uses API key authentication via the `key` query parameter rather than the `Authorization` header used by most other providers.
