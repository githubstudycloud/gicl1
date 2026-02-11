# Codex CLI 接口分析

## 基本信息

- **开发者**: OpenAI
- **平台**: macOS, Linux (Windows WSL)
- **认证**: ChatGPT 账户或 API Key
- **默认模型**: gpt-5.3-codex

## CLI 接口

### 基本命令

```bash
# 交互模式
codex

# 单次查询
codex "your prompt"

# 生成 TypeScript Schema
codex app-server generate-ts --out ./schemas

# 生成 JSON Schema
codex app-server generate-json-schema --out ./schemas
```

### 配置文件

位置: `~/.codex/config.toml`

```toml
model = "gpt-5.3-codex"

[mcp]
servers = [
  { name = "filesystem", command = "npx", args = ["-y", "@modelcontextprotocol/server-filesystem", "/path"] }
]
```

## App Server 协议

Codex 使用 **JSON-RPC 2.0** 协议（省略 `"jsonrpc":"2.0"` 头）。

### 通信方式

- **传输**: JSONL over stdio
- **方向**: 双向通信
- **用途**: 支持 IDE 扩展等富客户端

### 任务结构

```json
{
  "tasks": [
    {
      "id": "task_xxx",
      "url": "https://...",
      "title": "Task Title",
      "status": "completed",
      "updated_at": "2025-01-15T10:00:00Z",
      "environment_id": "env_xxx",
      "environment_label": "production",
      "summary": "Task summary",
      "is_review": false,
      "attempt_total": 1
    }
  ],
  "cursor": "next_page_cursor"
}
```

## 底层 API 格式

Codex 使用 **OpenAI Chat Completions API**。

### 请求格式

```json
{
  "model": "gpt-5.3-codex",
  "messages": [
    {"role": "system", "content": "You are a helpful coding assistant."},
    {"role": "user", "content": "Write a hello world in Python"}
  ],
  "temperature": 0.7,
  "max_tokens": 4096
}
```

### 响应格式

```json
{
  "id": "chatcmpl-xxx",
  "object": "chat.completion",
  "model": "gpt-5.3-codex",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "print('Hello, World!')"
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 20,
    "completion_tokens": 10,
    "total_tokens": 30
  }
}
```

### 工具调用格式

```json
{
  "model": "gpt-5.3-codex",
  "messages": [...],
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "run_command",
        "description": "Execute a shell command",
        "parameters": {
          "type": "object",
          "properties": {
            "command": {"type": "string"}
          },
          "required": ["command"]
        }
      }
    }
  ]
}
```

### 工具调用响应

```json
{
  "choices": [
    {
      "message": {
        "role": "assistant",
        "tool_calls": [
          {
            "id": "call_xxx",
            "type": "function",
            "function": {
              "name": "run_command",
              "arguments": "{\"command\": \"ls -la\"}"
            }
          }
        ]
      }
    }
  ]
}
```

### 工具结果返回

```json
{
  "role": "tool",
  "tool_call_id": "call_xxx",
  "content": "total 8\ndrwxr-xr-x  2 user user 4096 Jan 15 10:00 ."
}
```

## MCP 支持

配置在 `config.toml`:

```toml
[[mcp.servers]]
name = "my-server"
type = "stdio"
command = "node"
args = ["server.js"]

[[mcp.servers]]
name = "remote-server"
type = "http"
url = "https://mcp.example.com"
```

支持:
- **STDIO 服务器**: 本地进程
- **Streamable HTTP 服务器**: 远程访问

## 结构化输出

```json
{
  "model": "gpt-5.3-codex",
  "response_format": {
    "type": "json_schema",
    "json_schema": {
      "name": "output",
      "strict": true,
      "schema": {
        "type": "object",
        "properties": {...}
      }
    }
  }
}
```
