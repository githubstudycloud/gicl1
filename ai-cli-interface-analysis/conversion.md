# 接口转换方案

本文档介绍如何在不同 AI API 格式之间进行转换。

## 转换架构图

```
┌─────────────┐     ┌──────────────────┐     ┌─────────────┐
│  OpenAI     │     │                  │     │  Anthropic  │
│  Format     │────▶│  Universal       │────▶│  Format     │
└─────────────┘     │  Intermediate    │     └─────────────┘
                    │  Format (UIF)    │
┌─────────────┐     │                  │     ┌─────────────┐
│  Gemini     │────▶│                  │────▶│  OpenAI     │
│  Format     │     └──────────────────┘     │  Compatible │
└─────────────┘                              └─────────────┘
```

---

## 1. OpenAI → Anthropic 转换

### 消息转换

```javascript
function openaiToAnthropic(openaiRequest) {
  const messages = [];
  let systemPrompt = null;

  for (const msg of openaiRequest.messages) {
    if (msg.role === 'system') {
      systemPrompt = msg.content;
      continue;
    }

    if (msg.role === 'user') {
      messages.push({
        role: 'user',
        content: Array.isArray(msg.content)
          ? msg.content.map(convertContentBlock)
          : [{ type: 'text', text: msg.content }]
      });
    }

    if (msg.role === 'assistant') {
      messages.push({
        role: 'assistant',
        content: [{ type: 'text', text: msg.content }]
      });
    }

    if (msg.role === 'tool') {
      // 工具结果需要合并到 user 消息
      messages.push({
        role: 'user',
        content: [{
          type: 'tool_result',
          tool_use_id: msg.tool_call_id,
          content: msg.content
        }]
      });
    }
  }

  return {
    model: mapModelName(openaiRequest.model),
    system: systemPrompt,
    messages,
    max_tokens: openaiRequest.max_tokens || 4096,
    temperature: openaiRequest.temperature,
    tools: openaiRequest.tools?.map(convertTool)
  };
}

function convertTool(openaiTool) {
  return {
    name: openaiTool.function.name,
    description: openaiTool.function.description,
    input_schema: openaiTool.function.parameters
  };
}
```

### 响应转换

```javascript
function anthropicToOpenai(anthropicResponse) {
  const content = anthropicResponse.content
    .filter(c => c.type === 'text')
    .map(c => c.text)
    .join('');

  const toolCalls = anthropicResponse.content
    .filter(c => c.type === 'tool_use')
    .map(c => ({
      id: c.id,
      type: 'function',
      function: {
        name: c.name,
        arguments: JSON.stringify(c.input)
      }
    }));

  return {
    id: anthropicResponse.id,
    object: 'chat.completion',
    model: anthropicResponse.model,
    choices: [{
      index: 0,
      message: {
        role: 'assistant',
        content: content || null,
        tool_calls: toolCalls.length > 0 ? toolCalls : undefined
      },
      finish_reason: mapStopReason(anthropicResponse.stop_reason)
    }],
    usage: {
      prompt_tokens: anthropicResponse.usage.input_tokens,
      completion_tokens: anthropicResponse.usage.output_tokens,
      total_tokens: anthropicResponse.usage.input_tokens + anthropicResponse.usage.output_tokens
    }
  };
}
```

---

## 2. OpenAI → Gemini 转换

### 消息转换

```javascript
function openaiToGemini(openaiRequest) {
  const contents = [];
  let systemInstruction = null;

  for (const msg of openaiRequest.messages) {
    if (msg.role === 'system') {
      systemInstruction = { parts: [{ text: msg.content }] };
      continue;
    }

    const role = msg.role === 'assistant' ? 'model' : 'user';

    if (msg.role === 'tool') {
      contents.push({
        role: 'user',
        parts: [{
          functionResponse: {
            name: msg.name || 'tool_response',
            response: JSON.parse(msg.content)
          }
        }]
      });
      continue;
    }

    contents.push({
      role,
      parts: Array.isArray(msg.content)
        ? msg.content.map(convertToGeminiPart)
        : [{ text: msg.content }]
    });
  }

  return {
    contents,
    systemInstruction,
    generationConfig: {
      temperature: openaiRequest.temperature,
      maxOutputTokens: openaiRequest.max_tokens
    },
    tools: openaiRequest.tools ? [{
      functionDeclarations: openaiRequest.tools.map(t => ({
        name: t.function.name,
        description: t.function.description,
        parameters: t.function.parameters
      }))
    }] : undefined
  };
}
```

### 响应转换

```javascript
function geminiToOpenai(geminiResponse) {
  const candidate = geminiResponse.candidates[0];
  const parts = candidate.content.parts;

  const textContent = parts
    .filter(p => p.text)
    .map(p => p.text)
    .join('');

  const toolCalls = parts
    .filter(p => p.functionCall)
    .map((p, i) => ({
      id: `call_${i}`,
      type: 'function',
      function: {
        name: p.functionCall.name,
        arguments: JSON.stringify(p.functionCall.args)
      }
    }));

  return {
    id: 'gemini-' + Date.now(),
    object: 'chat.completion',
    model: 'gemini',
    choices: [{
      index: 0,
      message: {
        role: 'assistant',
        content: textContent || null,
        tool_calls: toolCalls.length > 0 ? toolCalls : undefined
      },
      finish_reason: mapGeminiFinishReason(candidate.finishReason)
    }],
    usage: {
      prompt_tokens: geminiResponse.usageMetadata?.promptTokenCount || 0,
      completion_tokens: geminiResponse.usageMetadata?.candidatesTokenCount || 0,
      total_tokens: geminiResponse.usageMetadata?.totalTokenCount || 0
    }
  };
}
```

---

## 3. Anthropic → Gemini 转换

```javascript
function anthropicToGemini(anthropicRequest) {
  const contents = anthropicRequest.messages.map(msg => ({
    role: msg.role === 'assistant' ? 'model' : 'user',
    parts: msg.content.map(block => {
      if (block.type === 'text') return { text: block.text };
      if (block.type === 'image') {
        return {
          inlineData: {
            mimeType: block.source.media_type,
            data: block.source.data
          }
        };
      }
      if (block.type === 'tool_result') {
        return {
          functionResponse: {
            name: 'tool_' + block.tool_use_id,
            response: JSON.parse(block.content)
          }
        };
      }
    })
  }));

  return {
    contents,
    systemInstruction: anthropicRequest.system
      ? { parts: [{ text: anthropicRequest.system }] }
      : undefined,
    generationConfig: {
      temperature: anthropicRequest.temperature,
      maxOutputTokens: anthropicRequest.max_tokens
    },
    tools: anthropicRequest.tools ? [{
      functionDeclarations: anthropicRequest.tools.map(t => ({
        name: t.name,
        description: t.description,
        parameters: t.input_schema
      }))
    }] : undefined
  };
}
```

---

## 4. 通用中间格式 (UIF)

建议的统一中间格式，便于多向转换：

```typescript
interface UniversalMessage {
  role: 'system' | 'user' | 'assistant' | 'tool';
  content: UniversalContent[];
  toolCallId?: string;  // for tool results
}

interface UniversalContent {
  type: 'text' | 'image' | 'tool_call' | 'tool_result';
  text?: string;
  image?: {
    mimeType: string;
    data: string;  // base64
    url?: string;
  };
  toolCall?: {
    id: string;
    name: string;
    arguments: object;
  };
  toolResult?: {
    callId: string;
    content: string | object;
  };
}

interface UniversalRequest {
  model: string;
  messages: UniversalMessage[];
  systemPrompt?: string;
  temperature?: number;
  maxTokens?: number;
  tools?: UniversalTool[];
  stream?: boolean;
}

interface UniversalTool {
  name: string;
  description: string;
  parameters: JSONSchema;
}

interface UniversalResponse {
  id: string;
  model: string;
  content: UniversalContent[];
  finishReason: 'stop' | 'tool_calls' | 'length' | 'error';
  usage: {
    inputTokens: number;
    outputTokens: number;
  };
}
```

---

## 5. 现有转换工具/代理

### LiteLLM

```python
# pip install litellm

from litellm import completion

# 统一接口调用不同模型
response = completion(
    model="anthropic/claude-sonnet-4-20250514",  # 或 "gpt-4o", "gemini/gemini-pro"
    messages=[{"role": "user", "content": "Hello"}]
)
```

### OpenRouter

提供统一的 OpenAI 兼容 API，支持多种模型：

```bash
curl https://openrouter.ai/api/v1/chat/completions \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" \
  -d '{
    "model": "anthropic/claude-sonnet-4-20250514",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

### Claude Code OpenAI Gateway

Claude Code 内置 OpenAI 兼容网关：

```bash
# 使用 OpenAI SDK 调用 Claude Code
curl http://localhost:8080/v1/chat/completions \
  -d '{"model": "claude-sonnet-4-20250514", "messages": [...]}'
```

---

## 6. 流式转换

### OpenAI SSE → Anthropic SSE

```javascript
function* convertOpenAIStreamToAnthropic(openaiStream) {
  for (const chunk of openaiStream) {
    if (chunk === '[DONE]') {
      yield { type: 'message_stop' };
      return;
    }

    const data = JSON.parse(chunk);
    const delta = data.choices[0]?.delta;

    if (delta?.content) {
      yield {
        type: 'content_block_delta',
        delta: { type: 'text_delta', text: delta.content }
      };
    }

    if (delta?.tool_calls) {
      for (const tc of delta.tool_calls) {
        yield {
          type: 'content_block_delta',
          delta: {
            type: 'input_json_delta',
            partial_json: tc.function.arguments
          }
        };
      }
    }
  }
}
```

---

## 7. 最佳实践

### 转换时的注意事项

1. **模型名称映射**: 维护模型名称映射表
2. **Token 限制**: 不同模型有不同限制
3. **工具调用 ID**: Gemini 不使用 ID，需要生成
4. **System Prompt**: 位置和格式不同
5. **多模态**: 支持程度不同
6. **错误处理**: 错误格式各异

### 推荐架构

```
Client App
    │
    ▼
┌─────────────────┐
│  Adapter Layer  │  ◀── 统一接口
├─────────────────┤
│  OpenAI Adapter │
│  Claude Adapter │
│  Gemini Adapter │
└─────────────────┘
    │
    ▼
Various AI APIs
```

### 工具选择

| 场景 | 推荐方案 |
|------|----------|
| 简单转换 | 自写适配器 |
| 多模型切换 | LiteLLM |
| 生产环境 | OpenRouter / AWS Bedrock |
| 本地开发 | 自建转换层 |
