---
title: OpenCode
---

# OpenCode

OpenCode's paid offering is two lanes behind one credential, differing only by
base URL:

| Lane | Base URL | Billing | `x-opencode-session` |
|:---|:---|:---|:---|
| **Go** | `https://opencode.ai/zen/go/v1` | monthly subscription | required |
| **Zen** | `https://opencode.ai/zen/v1` | account balance | not required |

Both serve three API standards from the same base, so LLM-Rosetta exposes three
shims and defaults to the Go lane. Use Zen by overriding `base_url` on the
provider entry.

## Shims

| Shim Name | Base Type | Default Base URL | Credentials |
|:---|:---|:---|:---|
| `opencode--chat_completions` | `chat_completions` | `https://opencode.ai/zen/go/v1` | `OPENCODE_API_KEY` |
| `opencode--messages` | `messages` | `https://opencode.ai/zen/go/v1` | `OPENCODE_API_KEY` |
| `opencode--open_responses` | `open_responses` | `https://opencode.ai/zen/go/v1` | `OPENCODE_API_KEY` |

Each resolves to its format's path under the base: `/chat/completions`,
`/messages` (the key is sent as `x-api-key`), and `/responses`. Not every model
is on every endpoint — the [Go docs](https://opencode.ai/docs/go/) list which
model is served where.

## Session header

The Go lane rejects an inference request that carries no `x-opencode-session`
with `400 MissingSessionID`; the upstream uses it to route the request and to
keep a prompt-cache-affine backend for the conversation. The shims declare
`connection.session_header: x-opencode-session`, and the gateway fills it:

- by default with a value **derived per conversation** — a hash of the same
  affinity identity used for key selection (client key plus the message prefix),
  so every turn of a conversation reuses one session and a different
  conversation gets a different one;
- or with the client's own value, forwarded verbatim, when the request already
  carries that header — handy for clients such as Codex that send a native
  session id.

Zen ignores the header.

## Gateway configuration

Go lane (default):

```jsonc
{
  "providers": {
    "opencode": { "shim": "opencode--chat_completions" }
  },
  "models": {
    "glm-5.3": "opencode"
  }
}
```

Zen lane — the same shim with a `base_url` override:

```jsonc
{
  "providers": {
    "opencode-zen": {
      "shim": "opencode--chat_completions",
      "base_url": "https://opencode.ai/zen/v1"
    }
  },
  "models": {
    "glm-5.3": "opencode-zen"
  }
}
```

Set `OPENCODE_API_KEY` in the gateway's environment — the credential is the same
for both lanes.

## Free lane

The keyless `/zen/v1` free pool is **not** wired up: the upstream restricts it to
its own desktop client, and it needs a policy decision before any work — see
[issue #884](https://github.com/Oaklight/llm-rosetta/issues/884). The Go
subscription does grant the two *Preview Free* models through the paid lane.
