---
title: OpenCode
---

# OpenCode

OpenCode 的付费服务是同一个凭据下的两条通道，只有 base URL 不同：

| 通道 | Base URL | 计费 | `x-opencode-session` |
|:---|:---|:---|:---|
| **Go** | `https://opencode.ai/zen/go/v1` | 月度订阅 | 必需 |
| **Zen** | `https://opencode.ai/zen/v1` | 账户余额 | 不要求 |

两条通道在同一个 base 下都提供三种 API 标准，因此 LLM-Rosetta 暴露三个 shim，
默认走 Go 通道。要用 Zen，在 provider 条目上覆盖 `base_url` 即可。

## Shims

| Shim 名称 | Base 类型 | 默认 Base URL | 凭据 |
|:---|:---|:---|:---|
| `opencode--chat_completions` | `chat_completions` | `https://opencode.ai/zen/go/v1` | `OPENCODE_API_KEY` |
| `opencode--messages` | `messages` | `https://opencode.ai/zen/go/v1` | `OPENCODE_API_KEY` |
| `opencode--open_responses` | `open_responses` | `https://opencode.ai/zen/go/v1` | `OPENCODE_API_KEY` |

三者分别解析到 base 下各自格式的路径：`/chat/completions`、`/messages`
（key 通过 `x-api-key` 发送）和 `/responses`。并非每个模型都在每个端点上——
具体哪个模型走哪个端点见 [Go 文档](https://opencode.ai/docs/go/)。

## Session 头

Go 通道会以 `400 MissingSessionID` 拒绝任何不带 `x-opencode-session` 的推理请求；
上游用它来路由，并为该会话绑定一个对 prompt cache 友好的后端。这些 shim 声明
`connection.session_header: x-opencode-session`，由网关填充：

- 默认填充一个**按会话派生**的值——对 key 选择所用的同一个 affinity identity
  （客户端 key 加消息前缀）做哈希，因此同一会话的每一轮都复用同一个 session，
  不同会话则得到不同的值；
- 若请求本身已带该头，则原样透传客户端的值——对会发送原生 session id 的客户端
  （如 Codex）很有用。

Zen 会忽略该头。

## 网关配置

Go 通道（默认）：

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

Zen 通道——同一个 shim 覆盖 `base_url`：

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

在网关进程的环境里设置 `OPENCODE_API_KEY`——两条通道用的是同一个凭据。

## 免费通道

keyless 的 `/zen/v1` 免费池**没有**接入：上游把它限制在自家桌面客户端内，
且在做任何工作前需要一次政策决策——见
[issue #884](https://github.com/Oaklight/llm-rosetta/issues/884)。Go 订阅本身
通过付费通道确实提供了两个 *Preview Free* 模型。
