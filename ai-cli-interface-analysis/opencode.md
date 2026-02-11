# OpenCode 接口分析

## 基本信息

- **开发者**: 社区开源
- **语言**: Go (Bubble Tea TUI)
- **仓库**: [opencode-ai/opencode](https://github.com/opencode-ai/opencode)
- **存储**: SQLite

## 支持的 Provider

| Provider | 说明 |
|----------|------|
| OpenAI | 官方 API |
| Anthropic | Claude 系列 |
| Google Gemini | Gemini 系列 |
| AWS Bedrock | 云端托管 |
| Groq | 高速推理 |
| Azure OpenAI | Azure 托管 |
| OpenRouter | 聚合网关 |
| OpenAI Compatible | 自定义兼容端点 |

## CLI 接口

### 基本命令

```bash
# 交互 TUI 模式
opencode

# 单次查询
opencode -p "your prompt"

# JSON 输出
opencode -p "prompt" -f json

# 启动 HTTP 服务
opencode serve
opencode serve --port 4096

# 导出会话
opencode export --session <id>
```

### 配置文件

位置优先级:
1. 命令行参数
2. 项目目录 `.opencode/config.json`
3. 用户目录配置

支持格式: **JSON** / **JSONC**

### 配置示例

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "litevllm": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "LiteVLLM GLM",
      "options": {
        "baseURL": "https://ai.example.com/v1",
        "apiKey": "{env:LLM_API_KEY}"
      },
      "models": {
        "glm-4.7": {
          "name": "GLM-4.7",
          "limit": {
            "context": 131072,
            "output": 8192
          }
        }
      }
    }
  },
  "model": "litevllm/glm-4.7"
}
```

### 模型格式

模型使用 `provider/model` 格式:
- `openai/gpt-4o`
- `anthropic/claude-sonnet-4-20250514`
- `litevllm/glm-4.7`

## API 接口格式

OpenCode 主要使用 **OpenAI 兼容格式**。

### HTTP Server API

启动服务: `opencode serve`

### ACP (Agent Client Protocol)

通过 stdin/stdout 使用 nd-JSON 通信:

```bash
opencode acp
```

### 请求格式 (OpenAI 兼容)

```json
{
  "model": "gpt-4o",
  "messages": [
    {"role": "system", "content": "You are a coding assistant."},
    {"role": "user", "content": "Help me write a function"}
  ],
  "temperature": 0.7,
  "stream": true
}
```

### 流式响应格式

```
data: {"id":"chatcmpl-xxx","choices":[{"delta":{"content":"Hello"}}]}
data: {"id":"chatcmpl-xxx","choices":[{"delta":{"content":" World"}}]}
data: [DONE]
```

## 输出格式

### 非交互模式输出

| 格式 | 说明 |
|------|------|
| `text` | 纯文本输出 |
| `json` | 结构化 JSON |

### JSON 输出结构

```json
{
  "response": "Assistant's response",
  "model": "openai/gpt-4o",
  "usage": {
    "prompt_tokens": 100,
    "completion_tokens": 50,
    "total_tokens": 150
  },
  "session_id": "sess_xxx"
}
```

## 会话管理

### 导出会话

```bash
opencode export --session <session_id> --format json
```

### 导入会话

```bash
opencode import --file session.json
```

支持从 OpenCode share URL 导入。

## 工具调用

OpenCode 支持工具集成:

- **命令执行**: Shell 命令
- **代码修改**: 文件编辑
- **LSP 集成**: 语言服务器协议

### 工具格式 (OpenAI 兼容)

```json
{
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "execute_command",
        "description": "Run a shell command",
        "parameters": {
          "type": "object",
          "properties": {
            "command": {"type": "string"}
          }
        }
      }
    }
  ]
}
```

## 特色功能

- **Vim-like 编辑器**: 内置 Vim 风格编辑
- **SQLite 持久化**: 会话本地存储
- **LSP 集成**: 代码智能支持
- **多 Provider**: 灵活切换模型提供商
