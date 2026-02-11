# Claude Code 接口分析

## 基本信息

- **开发者**: Anthropic
- **安装**: `npm install -g @anthropic-ai/claude-code`
- **认证**: Claude Pro/Max 订阅

## CLI 接口

### 基本命令

```bash
# 单次查询
claude -p "your prompt"

# 继续上次会话
claude -c

# JSON 输出
claude -p "query" --output-format json

# 指定自定义代理
claude --agents '{"name": {...}}'
```

### 输出格式

支持 `--output-format` 参数:
- `text` (默认)
- `json` - 结构化 JSON 输出
- `stream-json` - 流式 JSON

## API 接口格式

Claude Code 底层使用 **Anthropic Messages API**。

### 请求格式

```json
{
  "model": "claude-sonnet-4-20250514",
  "max_tokens": 1024,
  "system": "You are a helpful assistant.",
  "messages": [
    {
      "role": "user",
      "content": "Hello, Claude!"
    }
  ]
}
```

### 响应格式

```json
{
  "id": "msg_xxx",
  "type": "message",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "Hello! How can I help you today?"
    }
  ],
  "model": "claude-sonnet-4-20250514",
  "stop_reason": "end_turn",
  "usage": {
    "input_tokens": 10,
    "output_tokens": 20
  }
}
```

### 工具调用格式

```json
{
  "model": "claude-sonnet-4-20250514",
  "max_tokens": 1024,
  "tools": [
    {
      "name": "get_weather",
      "description": "Get current weather",
      "input_schema": {
        "type": "object",
        "properties": {
          "location": {"type": "string"}
        },
        "required": ["location"]
      }
    }
  ],
  "messages": [
    {"role": "user", "content": "What's the weather in Tokyo?"}
  ]
}
```

### 工具调用响应

```json
{
  "content": [
    {
      "type": "tool_use",
      "id": "toolu_xxx",
      "name": "get_weather",
      "input": {"location": "Tokyo"}
    }
  ]
}
```

### 工具结果返回

```json
{
  "role": "user",
  "content": [
    {
      "type": "tool_result",
      "tool_use_id": "toolu_xxx",
      "content": "{\"temperature\": 22, \"condition\": \"sunny\"}"
    }
  ]
}
```

## OpenAI 兼容网关

Claude Code 提供 OpenAI 兼容 API 网关:

- **端点**: `/v1/chat/completions`
- **端点**: `/v1/models`

这允许使用 OpenAI SDK 调用 Claude Code。

## MCP 支持

支持 Model Context Protocol (MCP) 服务器:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path"]
    }
  }
}
```

## 结构化输出

使用 `strict: true` 启用严格 JSON Schema 验证:

```json
{
  "tools": [
    {
      "name": "extract_data",
      "strict": true,
      "input_schema": {
        "type": "object",
        "properties": {...},
        "additionalProperties": false
      }
    }
  ]
}
```

需要 Beta Header: `anthropic-beta: structured-outputs-2025-11-13`
