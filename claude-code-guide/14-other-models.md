# 接入其他大模型

## 概述

Claude Code 可以通过代理服务连接到其他 LLM，包括：

- OpenAI (GPT-4, GPT-4o)
- Google Gemini
- 本地模型 (Llama, Qwen, DeepSeek)
- 其他 API 兼容服务

## 方法 1: claude-code-proxy

### 介绍

[claude-code-proxy](https://github.com/1rgs/claude-code-proxy) 是一个代理服务，让 Claude Code 使用 OpenAI、Gemini 或其他 LLM。

### 安装

```bash
# 克隆仓库
git clone https://github.com/1rgs/claude-code-proxy
cd claude-code-proxy

# 安装依赖
pip install -r requirements.txt
```

### 配置

创建 `.env` 文件：

```bash
# OpenAI
OPENAI_API_KEY=sk-xxx
BIG_MODEL=gpt-4.1
SMALL_MODEL=gpt-4.1-mini

# 或 Gemini
GOOGLE_API_KEY=xxx
BIG_MODEL=gemini-2.5-pro-preview
SMALL_MODEL=gemini-2.0-flash

# 或混合
BIG_MODEL=gpt-4.1           # 复杂任务
SMALL_MODEL=gemini-2.0-flash # 简单任务
```

### 启动代理

```bash
python proxy.py
# 代理运行在 http://localhost:8080
```

### 配置 Claude Code

```bash
export ANTHROPIC_BASE_URL="http://localhost:8080"
export ANTHROPIC_API_KEY="any-string"  # 代理不验证

claude
```

或在 `settings.json`:

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "http://localhost:8080",
    "ANTHROPIC_API_KEY": "placeholder"
  }
}
```

## 方法 2: LiteLLM 代理

### 安装 LiteLLM

```bash
pip install litellm[proxy]
```

### 配置文件

`litellm_config.yaml`:

```yaml
model_list:
  - model_name: claude-sonnet-4-20250514
    litellm_params:
      model: gpt-4o
      api_key: sk-xxx

  - model_name: claude-haiku-3-5-20241022
    litellm_params:
      model: gpt-4o-mini
      api_key: sk-xxx

  - model_name: claude-opus-4-5-20251101
    litellm_params:
      model: o1-preview
      api_key: sk-xxx
```

### 启动

```bash
litellm --config litellm_config.yaml --port 8000
```

### 使用

```bash
export ANTHROPIC_BASE_URL="http://localhost:8000"
claude
```

## 方法 3: 本地模型

### 使用 Ollama

```bash
# 安装 Ollama
curl -fsSL https://ollama.com/install.sh | sh

# 拉取模型
ollama pull qwen2.5-coder:32b
ollama pull deepseek-coder-v2:16b

# 启动服务
ollama serve
```

### 配置 LiteLLM 连接 Ollama

```yaml
model_list:
  - model_name: claude-sonnet-4-20250514
    litellm_params:
      model: ollama/qwen2.5-coder:32b
      api_base: http://localhost:11434

  - model_name: claude-haiku-3-5-20241022
    litellm_params:
      model: ollama/deepseek-coder-v2:16b
      api_base: http://localhost:11434
```

### 使用 vLLM

```bash
# 安装 vLLM
pip install vllm

# 启动服务
python -m vllm.entrypoints.openai.api_server \
  --model Qwen/Qwen2.5-Coder-32B-Instruct \
  --port 8000 \
  --api-key token-abc123
```

### 使用 llama.cpp

```bash
# 编译 llama.cpp
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp && make

# 启动服务器
./server -m models/qwen2.5-coder-32b.gguf --port 8080
```

## 方法 4: OpenRouter

[OpenRouter](https://openrouter.ai/) 提供统一 API 访问多个模型。

### 配置

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://openrouter.ai/api/v1",
    "ANTHROPIC_API_KEY": "sk-or-xxx"
  }
}
```

### 模型映射

在 OpenRouter 中，模型名会自动映射。

## 方法 5: Kong AI Gateway

企业级方案，支持路由、监控、限流。

### 配置

```yaml
services:
  - name: claude-to-gemini
    url: https://generativelanguage.googleapis.com
    routes:
      - name: claude-route
        paths:
          - /v1/messages
    plugins:
      - name: ai-proxy
        config:
          route_type: llm/v1/chat
          auth:
            header_name: x-goog-api-key
            header_value: ${GOOGLE_API_KEY}
          model:
            provider: gemini
            name: gemini-2.0-flash
          llm_format: anthropic  # 关键：接受 Claude 格式
```

## 模型映射建议

| Claude 模型 | OpenAI 替代 | Gemini 替代 | 本地替代 |
|-------------|-------------|-------------|----------|
| opus | o1-preview | gemini-2.5-pro | qwen2.5-coder-32b |
| sonnet | gpt-4o | gemini-2.0-flash | qwen2.5-coder-14b |
| haiku | gpt-4o-mini | gemini-2.0-flash-lite | deepseek-coder-v2 |

## 完整配置示例

### OpenAI 后端

`.claude/settings.local.json`:

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "http://localhost:8080",
    "ANTHROPIC_API_KEY": "placeholder"
  }
}
```

`proxy/.env`:

```bash
OPENAI_API_KEY=sk-xxx
BIG_MODEL=gpt-4o
SMALL_MODEL=gpt-4o-mini
```

### Gemini 后端

```bash
GOOGLE_API_KEY=xxx
BIG_MODEL=gemini-2.5-pro-preview
SMALL_MODEL=gemini-2.0-flash
```

### 混合后端

```yaml
# litellm_config.yaml
model_list:
  # 复杂任务用 OpenAI
  - model_name: claude-opus-4-5-20251101
    litellm_params:
      model: o1-preview
      api_key: ${OPENAI_API_KEY}

  # 日常任务用 Gemini（便宜）
  - model_name: claude-sonnet-4-20250514
    litellm_params:
      model: gemini/gemini-2.0-flash
      api_key: ${GOOGLE_API_KEY}

  # 简单任务用本地（免费）
  - model_name: claude-haiku-3-5-20241022
    litellm_params:
      model: ollama/qwen2.5-coder:7b
      api_base: http://localhost:11434
```

## 注意事项

### 兼容性

- 不是所有功能都完全兼容
- 工具调用格式可能有差异
- 某些高级功能可能不支持

### 性能

- 本地模型可能较慢
- 代理增加延迟
- 考虑网络稳定性

### 成本

| 方案 | 成本 | 速度 | 隐私 |
|------|------|------|------|
| 官方 Claude | $$ | 快 | 云端 |
| OpenAI | $$ | 快 | 云端 |
| Gemini | $ | 快 | 云端 |
| 本地模型 | 免费 | 慢 | 本地 |

## 替代工具: OpenCode

如果需要原生支持多模型，可以考虑 [OpenCode](https://opencode.ai/):

```bash
# 安装
npm install -g opencode

# 支持 75+ 模型
opencode --model gpt-4o
opencode --model gemini-2.0-flash
opencode --model ollama/qwen2.5-coder
```

OpenCode 与 Claude Code 功能类似，但原生支持多种 LLM。
