# Gemini CLI 接口分析

## 基本信息

- **开发者**: Google
- **仓库**: [google-gemini/gemini-cli](https://github.com/google-gemini/gemini-cli)
- **免费额度**: 60 请求/分钟, 1000 请求/天
- **默认模型**: Gemini 3 系列
- **上下文窗口**: 1M tokens

## CLI 接口

### 基本命令

```bash
# 交互模式
gemini

# 单次查询
gemini "your prompt"

# JSON 输出
gemini --output-format json "your prompt"

# 流式 JSON 输出
gemini --output-format stream-json "your prompt"
```

### 输出格式

| 格式 | 说明 |
|------|------|
| `text` | 默认文本输出 |
| `json` | 结构化 JSON |
| `stream-json` | 换行分隔的 JSON 事件流 |

### JSON 输出结构

```json
{
  "response": "Model's response text",
  "stats": {
    "duration": 1234,
    "turns": 1,
    "tool_calls": 0
  },
  "error": null
}
```

## API 接口格式

Gemini CLI 使用 **Gemini API**。

### 核心概念

- **Content**: 消息的顶层容器（用户或模型的一个轮次）
- **Part**: 支持多模态，一个 Content 可包含多个 Part

### 请求格式

```json
{
  "contents": [
    {
      "role": "user",
      "parts": [
        {"text": "Hello, Gemini!"}
      ]
    }
  ],
  "generationConfig": {
    "temperature": 0.7,
    "maxOutputTokens": 8192
  }
}
```

### 多模态请求

```json
{
  "contents": [
    {
      "role": "user",
      "parts": [
        {"text": "Describe this image"},
        {
          "inlineData": {
            "mimeType": "image/jpeg",
            "data": "base64_encoded_image"
          }
        }
      ]
    }
  ]
}
```

### 响应格式

```json
{
  "candidates": [
    {
      "content": {
        "role": "model",
        "parts": [
          {"text": "Hello! How can I help you today?"}
        ]
      },
      "finishReason": "STOP",
      "safetyRatings": [...]
    }
  ],
  "usageMetadata": {
    "promptTokenCount": 10,
    "candidatesTokenCount": 15,
    "totalTokenCount": 25
  }
}
```

## 端点类型

### 1. 标准生成 (generateContent)

```
POST /v1/models/{model}:generateContent
```

同步返回完整响应。

### 2. 流式生成 (streamGenerateContent)

```
POST /v1/models/{model}:streamGenerateContent
```

使用 Server-Sent Events (SSE) 推送响应块。

## 工具调用格式

### 定义工具

```json
{
  "contents": [...],
  "tools": [
    {
      "functionDeclarations": [
        {
          "name": "get_weather",
          "description": "Get weather for a location",
          "parameters": {
            "type": "object",
            "properties": {
              "location": {
                "type": "string",
                "description": "City name"
              }
            },
            "required": ["location"]
          }
        }
      ]
    }
  ]
}
```

### 工具调用响应

```json
{
  "candidates": [
    {
      "content": {
        "role": "model",
        "parts": [
          {
            "functionCall": {
              "name": "get_weather",
              "args": {"location": "Tokyo"}
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
  "contents": [
    {"role": "user", "parts": [{"text": "What's the weather?"}]},
    {
      "role": "model",
      "parts": [{"functionCall": {"name": "get_weather", "args": {"location": "Tokyo"}}}]
    },
    {
      "role": "user",
      "parts": [
        {
          "functionResponse": {
            "name": "get_weather",
            "response": {"temperature": 22, "condition": "sunny"}
          }
        }
      ]
    }
  ]
}
```

## 内置工具

- **Google Search**: 搜索接地
- **File Operations**: 文件操作
- **Shell Commands**: 命令执行
- **Web Fetching**: 网页获取

## MCP 支持

Gemini CLI 支持 MCP 服务器进行自定义集成。

## ReAct 循环

Gemini CLI 使用 **Reason and Act (ReAct)** 循环:
1. 推理当前情况
2. 选择合适的工具
3. 执行动作
4. 观察结果
5. 重复直到任务完成
