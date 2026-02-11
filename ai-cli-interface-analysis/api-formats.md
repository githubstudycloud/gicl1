# 通用 AI 会话 API 格式对比

## 三大主流 API 格式

### 1. OpenAI Chat Completions API

**端点**: `POST /v1/chat/completions`

这是目前最广泛采用的格式，许多第三方服务都提供兼容接口。

#### 请求结构

```json
{
  "model": "gpt-4o",
  "messages": [
    {"role": "system", "content": "System prompt"},
    {"role": "user", "content": "User message"},
    {"role": "assistant", "content": "Assistant response"},
    {"role": "tool", "tool_call_id": "xxx", "content": "Tool result"}
  ],
  "temperature": 0.7,
  "max_tokens": 4096,
  "stream": false,
  "tools": [...],
  "response_format": {"type": "json_object"}
}
```

#### 响应结构

```json
{
  "id": "chatcmpl-xxx",
  "object": "chat.completion",
  "created": 1234567890,
  "model": "gpt-4o",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Response text",
        "tool_calls": [...]
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 100,
    "completion_tokens": 50,
    "total_tokens": 150
  }
}
```

#### 工具调用

```json
{
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "function_name",
        "description": "Description",
        "parameters": {
          "type": "object",
          "properties": {...},
          "required": [...]
        }
      }
    }
  ]
}
```

#### 工具调用响应

```json
{
  "message": {
    "role": "assistant",
    "tool_calls": [
      {
        "id": "call_xxx",
        "type": "function",
        "function": {
          "name": "function_name",
          "arguments": "{\"arg\": \"value\"}"
        }
      }
    ]
  }
}
```

---

### 2. Anthropic Messages API

**端点**: `POST /v1/messages`

#### 请求结构

```json
{
  "model": "claude-sonnet-4-20250514",
  "max_tokens": 4096,
  "system": "System prompt",
  "messages": [
    {
      "role": "user",
      "content": [
        {"type": "text", "text": "User message"},
        {"type": "image", "source": {"type": "base64", "media_type": "image/png", "data": "..."}}
      ]
    },
    {
      "role": "assistant",
      "content": [
        {"type": "text", "text": "Response"}
      ]
    }
  ],
  "tools": [...],
  "temperature": 0.7
}
```

#### 响应结构

```json
{
  "id": "msg_xxx",
  "type": "message",
  "role": "assistant",
  "content": [
    {"type": "text", "text": "Response text"},
    {"type": "tool_use", "id": "toolu_xxx", "name": "func", "input": {...}}
  ],
  "model": "claude-sonnet-4-20250514",
  "stop_reason": "end_turn",
  "usage": {
    "input_tokens": 100,
    "output_tokens": 50
  }
}
```

#### 工具定义

```json
{
  "tools": [
    {
      "name": "function_name",
      "description": "Description",
      "input_schema": {
        "type": "object",
        "properties": {...},
        "required": [...]
      }
    }
  ]
}
```

#### 工具结果返回

```json
{
  "role": "user",
  "content": [
    {
      "type": "tool_result",
      "tool_use_id": "toolu_xxx",
      "content": "Result string or JSON"
    }
  ]
}
```

---

### 3. Google Gemini API

**端点**: `POST /v1/models/{model}:generateContent`

#### 请求结构

```json
{
  "contents": [
    {
      "role": "user",
      "parts": [
        {"text": "User message"},
        {"inlineData": {"mimeType": "image/jpeg", "data": "base64..."}}
      ]
    },
    {
      "role": "model",
      "parts": [{"text": "Model response"}]
    }
  ],
  "generationConfig": {
    "temperature": 0.7,
    "maxOutputTokens": 8192
  },
  "tools": [...]
}
```

#### 响应结构

```json
{
  "candidates": [
    {
      "content": {
        "role": "model",
        "parts": [{"text": "Response text"}]
      },
      "finishReason": "STOP"
    }
  ],
  "usageMetadata": {
    "promptTokenCount": 100,
    "candidatesTokenCount": 50,
    "totalTokenCount": 150
  }
}
```

#### 工具定义

```json
{
  "tools": [
    {
      "functionDeclarations": [
        {
          "name": "function_name",
          "description": "Description",
          "parameters": {
            "type": "object",
            "properties": {...},
            "required": [...]
          }
        }
      ]
    }
  ]
}
```

#### 工具调用响应

```json
{
  "content": {
    "parts": [
      {
        "functionCall": {
          "name": "function_name",
          "args": {"arg": "value"}
        }
      }
    ]
  }
}
```

#### 工具结果返回

```json
{
  "role": "user",
  "parts": [
    {
      "functionResponse": {
        "name": "function_name",
        "response": {"result": "value"}
      }
    }
  ]
}
```

---

## 关键差异对比表

| 特性 | OpenAI | Anthropic | Gemini |
|------|--------|-----------|--------|
| 消息容器 | `messages[]` | `messages[]` | `contents[]` |
| 内容结构 | 字符串或数组 | Content Blocks 数组 | Parts 数组 |
| System Prompt | `role: "system"` | 单独 `system` 字段 | `systemInstruction` |
| 助手角色名 | `assistant` | `assistant` | `model` |
| 工具角色 | `role: "tool"` | 在 user 消息中 | 在 user 消息中 |
| 工具定义 | `function` 包装 | 直接定义 | `functionDeclarations` |
| 工具调用 ID | `tool_call_id` | `tool_use_id` | 无 (用 name) |
| Token 统计 | `usage.xxx_tokens` | `usage.xxx_tokens` | `usageMetadata.xxxTokenCount` |
| 停止原因 | `finish_reason` | `stop_reason` | `finishReason` |

---

## 多模态支持对比

| 格式 | 文本 | 图片 | 视频 | 音频 | 文件 |
|------|------|------|------|------|------|
| OpenAI | ✓ | ✓ | ✓ | ✓ | ✓ |
| Anthropic | ✓ | ✓ | ✗ | ✗ | ✓ (PDF) |
| Gemini | ✓ | ✓ | ✓ | ✓ | ✓ |

---

## 流式响应对比

### OpenAI (SSE)

```
data: {"choices":[{"delta":{"content":"Hello"}}]}
data: {"choices":[{"delta":{"content":" World"}}]}
data: [DONE]
```

### Anthropic (SSE)

```
event: content_block_delta
data: {"type":"content_block_delta","delta":{"type":"text_delta","text":"Hello"}}

event: message_stop
data: {"type":"message_stop"}
```

### Gemini (SSE)

```
data: {"candidates":[{"content":{"parts":[{"text":"Hello"}]}}]}
data: {"candidates":[{"content":{"parts":[{"text":" World"}]}}]}
```
