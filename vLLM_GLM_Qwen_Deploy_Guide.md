# vLLM 部署 GLM-4.7 & Qwen3-235B-A22B 工具调用完整指南

## 目录
1. [环境要求](#环境要求)
2. [GLM-4.7 部署配置](#glm-47-部署配置)
3. [Qwen3-235B-A22B 部署配置](#qwen3-235b-a22b-部署配置)
4. [核心参数详解](#核心参数详解)
5. [UFW 防火墙配置](#ufw-防火墙配置)
6. [MCP 集成说明](#mcp-集成说明)
7. [常见问题排查](#常见问题排查)

---

## 环境要求

### 硬件要求

| 模型 | 最低显存 | 推荐配置 | 说明 |
|------|---------|---------|------|
| GLM-4.7 (FP8) | 4x A100 80G | 4x H100 | - |
| GLM-4.7 (原始) | 8x A100 80G | 8x H100 | - |
| Qwen3-235B-A22B | 8x A100 80G | 8x H100 | MoE 模型，总参数 235B，激活 22B |
| Qwen3-235B-A22B-FP8 | 4x H100 80G | 8x H100 | FP8 量化版本 |
| Qwen3-235B-A22B-w8a8 | 4x A100 80G | 8x A100 80G | INT8 量化版本 |

> **注意**: Qwen3-235B-A22B 是 MoE（Mixture of Experts）架构，总参数量 235B，但每次推理只激活 22B 参数，因此推理效率较高。

### 软件要求

```bash
# 安装最新 vLLM (需要 >= 0.8.5 版本)
pip install vllm --pre --extra-index-url https://wheels.vllm.ai/nightly

# 或者从源码安装（GLM-4.7 需要 nightly 版本）
pip install git+https://github.com/vllm-project/vllm.git

# 验证安装
python -c "import vllm; print(vllm.__version__)"

# 推荐版本: vllm >= 0.9.0 (支持 qwen3 reasoning parser)
```

---

## GLM-4.7 部署配置

### 基础部署命令

```bash
vllm serve zai-org/GLM-4.7 \
    --host 0.0.0.0 \
    --port 8000 \
    --tensor-parallel-size 4 \
    --enable-auto-tool-choice \
    --tool-call-parser glm47 \
    --reasoning-parser glm45 \
    --served-model-name glm-4.7
```

### FP8 量化版本（推荐，节省显存）

```bash
vllm serve zai-org/GLM-4.7-FP8 \
    --host 0.0.0.0 \
    --port 8000 \
    --tensor-parallel-size 4 \
    --speculative-config.method mtp \
    --speculative-config.num_speculative_tokens 1 \
    --enable-auto-tool-choice \
    --tool-call-parser glm47 \
    --reasoning-parser glm45 \
    --gpu-memory-utilization 0.9 \
    --max-model-len 32768 \
    --served-model-name glm-4.7-fp8
```

### 完整生产环境配置

```bash
#!/bin/bash
# glm47_serve.sh

export CUDA_VISIBLE_DEVICES=0,1,2,3

vllm serve zai-org/GLM-4.7-FP8 \
    --host 0.0.0.0 \
    --port 8000 \
    --tensor-parallel-size 4 \
    --enable-auto-tool-choice \
    --tool-call-parser glm47 \
    --reasoning-parser glm45 \
    --speculative-config.method mtp \
    --speculative-config.num_speculative_tokens 1 \
    --gpu-memory-utilization 0.9 \
    --max-model-len 32768 \
    --max-num-seqs 64 \
    --trust-remote-code \
    --disable-log-requests \
    --uvicorn-log-level warning \
    --served-model-name glm-4.7
```

---

## Qwen3-235B-A22B 部署配置

> **重要**: Qwen3-235B-A22B 是 MoE 架构模型（总参数 235B，激活参数 22B），需要较大的显存和多卡部署。

### 基础部署命令

```bash
vllm serve Qwen/Qwen3-235B-A22B \
    --host 0.0.0.0 \
    --port 8001 \
    --tensor-parallel-size 8 \
    --enable-auto-tool-choice \
    --tool-call-parser hermes \
    --reasoning-parser deepseek_r1 \
    --gpu-memory-utilization 0.9 \
    --max-model-len 32768 \
    --trust-remote-code \
    --served-model-name qwen3-235b
```

### 最新 Instruct 版本（2507，推荐）

```bash
vllm serve Qwen/Qwen3-235B-A22B-Instruct-2507 \
    --host 0.0.0.0 \
    --port 8001 \
    --tensor-parallel-size 8 \
    --enable-auto-tool-choice \
    --tool-call-parser hermes \
    --reasoning-parser deepseek_r1 \
    --gpu-memory-utilization 0.9 \
    --max-model-len 32768 \
    --trust-remote-code \
    --served-model-name qwen3-235b-instruct
```

### FP8 量化版本（节省显存）

```bash
vllm serve Qwen/Qwen3-235B-A22B-Instruct-2507-FP8 \
    --host 0.0.0.0 \
    --port 8001 \
    --tensor-parallel-size 4 \
    --enable-expert-parallel \
    --enable-auto-tool-choice \
    --tool-call-parser hermes \
    --reasoning-parser deepseek_r1 \
    --gpu-memory-utilization 0.9 \
    --max-model-len 32768 \
    --trust-remote-code \
    --served-model-name qwen3-235b-fp8
```

### Thinking 版本（支持深度推理）

```bash
vllm serve Qwen/Qwen3-235B-A22B-Thinking-2507 \
    --host 0.0.0.0 \
    --port 8001 \
    --tensor-parallel-size 8 \
    --enable-auto-tool-choice \
    --tool-call-parser hermes \
    --reasoning-parser deepseek_r1 \
    --gpu-memory-utilization 0.9 \
    --max-model-len 262144 \
    --trust-remote-code \
    --served-model-name qwen3-235b-thinking
```

### 使用 Expert Parallel（MoE 优化）

当 tensor-parallel 出现问题时，可以启用 expert parallel：

```bash
vllm serve Qwen/Qwen3-235B-A22B-Instruct-2507-FP8 \
    --host 0.0.0.0 \
    --port 8001 \
    --tensor-parallel-size 8 \
    --enable-expert-parallel \
    --enable-auto-tool-choice \
    --tool-call-parser hermes \
    --reasoning-parser deepseek_r1 \
    --gpu-memory-utilization 0.9 \
    --max-model-len 32768 \
    --trust-remote-code \
    --served-model-name qwen3-235b
```

### 完整生产环境配置

```bash
#!/bin/bash
# qwen3_235b_serve.sh

export CUDA_VISIBLE_DEVICES=0,1,2,3,4,5,6,7

vllm serve Qwen/Qwen3-235B-A22B-Instruct-2507 \
    --host 0.0.0.0 \
    --port 8001 \
    --tensor-parallel-size 8 \
    --enable-auto-tool-choice \
    --tool-call-parser hermes \
    --reasoning-parser deepseek_r1 \
    --gpu-memory-utilization 0.9 \
    --max-model-len 32768 \
    --max-num-seqs 64 \
    --trust-remote-code \
    --disable-log-requests \
    --uvicorn-log-level warning \
    --served-model-name qwen3-235b
```

### 使用 Qwen-Agent（官方推荐方式）

Qwen 官方推荐使用 Qwen-Agent 来获得最佳的工具调用体验：

```bash
# 安装 Qwen-Agent
pip install qwen-agent

# 禁用 vLLM 内置解析，让 Qwen-Agent 处理
vllm serve Qwen/Qwen3-235B-A22B-Instruct-2507 \
    --host 0.0.0.0 \
    --port 8001 \
    --tensor-parallel-size 8 \
    --gpu-memory-utilization 0.9 \
    --max-model-len 32768 \
    --trust-remote-code \
    --served-model-name qwen3-235b
```

---

## 核心参数详解

### 工具调用相关参数

| 参数 | 说明 | 可选值 |
|------|------|--------|
| `--enable-auto-tool-choice` | **必需**。启用自动工具选择，允许模型自主决定是否调用工具 | 无值，直接使用 |
| `--tool-call-parser` | **必需**。指定工具调用解析器，不同模型需要不同解析器 | `hermes`, `mistral`, `llama3_json`, `glm45`, `glm47`, `qwen3_coder`, `internlm` |
| `--chat-template` | 可选。指定聊天模板路径，用于处理工具消息 | 模板文件路径或 `tool_use` |
| `--tool-parser-plugin` | 可选。注册自定义工具解析器插件 | 插件文件绝对路径 |

### 推理/思考相关参数

| 参数 | 说明 | 可选值 |
|------|------|--------|
| `--reasoning-parser` | 推理内容解析器，用于处理模型的思考过程 | `deepseek_r1`, `glm45`, `qwen3` |
| `--enable-reasoning` | 启用推理模式 | 无值，直接使用 |

### 模型解析器对照表

| 模型系列 | tool-call-parser | reasoning-parser |
|---------|------------------|------------------|
| GLM-4.7 | `glm47` | `glm45` |
| GLM-4.5/4.6 | `glm45` | `glm45` |
| **Qwen3-235B-A22B** | `hermes` | `deepseek_r1` |
| Qwen3 (其他) | `hermes` | `deepseek_r1` 或 `qwen3` |
| Qwen3-Coder | `qwen3_coder` | `qwen3` |
| Qwen2.5 | `hermes` | 不需要 |
| Llama 3.x | `llama3_json` | 不需要 |
| Mistral | `mistral` | 不需要 |
| DeepSeek | `hermes` | `deepseek_r1` |

### MoE 专用参数

| 参数 | 说明 | 推荐值 |
|------|------|--------|
| `--enable-expert-parallel` | 启用专家并行（MoE 模型优化） | 当 TP 出现问题时使用 |

### 性能优化参数

| 参数 | 说明 | 推荐值 |
|------|------|--------|
| `--tensor-parallel-size` | 张量并行数，通常等于 GPU 数量 | Qwen3-235B: 8, GLM-4.7: 4 |
| `--gpu-memory-utilization` | GPU 显存利用率 | 0.85-0.95 |
| `--max-model-len` | 最大上下文长度 | 32768-262144 |
| `--max-num-seqs` | 最大并发序列数 | 32-256 |
| `--speculative-config.method` | 推测解码方法 | `mtp`（GLM专用） |
| `--speculative-config.num_speculative_tokens` | 推测 token 数 | 1（推荐） |

### 服务配置参数

| 参数 | 说明 | 推荐值 |
|------|------|--------|
| `--host` | 监听地址 | `0.0.0.0` |
| `--port` | 监听端口 | `8000` |
| `--served-model-name` | API 中显示的模型名称 | 自定义 |
| `--trust-remote-code` | 信任远程代码（部分模型需要） | 直接使用 |
| `--disable-log-requests` | 禁用请求日志（生产环境推荐） | 直接使用 |
| `--uvicorn-log-level` | 日志级别 | `warning` |

---

## UFW 防火墙配置

### 为什么 UFW 会导致卡顿？

1. **连接速率限制**：UFW 默认在 30 秒内超过 6 次连接会触发限制
2. **日志记录开销**：每个数据包的日志记录会消耗 CPU
3. **连接追踪表溢出**：大量并发连接可能耗尽 conntrack 表

### 推荐 UFW 配置

```bash
# 1. 首先备份当前规则
sudo cp /etc/ufw/user.rules /etc/ufw/user.rules.backup
sudo cp /etc/ufw/user6.rules /etc/ufw/user6.rules.backup

# 2. 重置 UFW（可选，谨慎操作）
# sudo ufw reset

# 3. 设置默认策略
sudo ufw default deny incoming
sudo ufw default allow outgoing

# 4. 允许 SSH（重要！防止锁死）
sudo ufw allow 22/tcp

# 5. 开放 vLLM API 端口（无速率限制）
sudo ufw allow 8000/tcp comment 'vLLM GLM-4.7 API'
sudo ufw allow 8001/tcp comment 'vLLM Qwen3-235B API'

# 6. 如果需要开放端口范围
sudo ufw allow 8000:8010/tcp comment 'vLLM API Range'

# 7. 允许本地回环
sudo ufw allow from 127.0.0.1

# 8. 允许局域网访问（可选）
sudo ufw allow from 192.168.0.0/16
sudo ufw allow from 10.0.0.0/8

# 9. 关闭日志（减少性能开销）
sudo ufw logging off

# 10. 启用 UFW
sudo ufw enable

# 11. 查看状态
sudo ufw status verbose
```

### 优化 conntrack 参数（解决高并发卡顿）

```bash
# 编辑 sysctl 配置
sudo nano /etc/sysctl.conf

# 添加以下内容
net.netfilter.nf_conntrack_max = 1048576
net.netfilter.nf_conntrack_tcp_timeout_established = 600
net.netfilter.nf_conntrack_tcp_timeout_time_wait = 30
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535

# 应用配置
sudo sysctl -p
```

### 检查 UFW 是否阻止连接

```bash
# 查看 UFW 日志
sudo tail -f /var/log/ufw.log

# 查看被阻止的连接
sudo grep -i "block" /var/log/ufw.log | tail -20

# 检查 conntrack 使用情况
cat /proc/sys/net/netfilter/nf_conntrack_count
cat /proc/sys/net/netfilter/nf_conntrack_max
```

### 完整 UFW 配置脚本

```bash
#!/bin/bash
# setup_ufw.sh - vLLM 服务器 UFW 配置脚本

set -e

echo "=== 配置 UFW 防火墙 ==="

# 备份
sudo cp /etc/ufw/user.rules /etc/ufw/user.rules.backup.$(date +%Y%m%d)

# 重置规则（谨慎！）
# sudo ufw --force reset

# 默认策略
sudo ufw default deny incoming
sudo ufw default allow outgoing

# SSH
sudo ufw allow 22/tcp

# vLLM API 端口
sudo ufw allow 8000/tcp comment 'vLLM-GLM47'
sudo ufw allow 8001/tcp comment 'vLLM-Qwen3-235B'

# 本地和局域网
sudo ufw allow from 127.0.0.1
sudo ufw allow from 192.168.0.0/16
sudo ufw allow from 10.0.0.0/8

# 关闭日志减少开销
sudo ufw logging off

# 启用
sudo ufw --force enable

echo "=== UFW 配置完成 ==="
sudo ufw status numbered
```

---

## MCP 集成说明

### 与 MCP 客户端集成

vLLM 启动后，提供 OpenAI 兼容的 API，可以直接与支持 OpenAI API 的 MCP 客户端集成。

### 配置示例

```json
{
  "mcpServers": {
    "your-mcp-server": {
      "command": "your-mcp-command"
    }
  },
  "llm": {
    "provider": "openai",
    "baseUrl": "http://localhost:8001/v1",
    "model": "qwen3-235b",
    "apiKey": "not-needed"
  }
}
```

### 测试工具调用

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:8001/v1",
    api_key="not-needed"
)

tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "获取指定城市的天气",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {
                        "type": "string",
                        "description": "城市名称"
                    }
                },
                "required": ["city"]
            }
        }
    }
]

response = client.chat.completions.create(
    model="qwen3-235b",  # 或 glm-4.7
    messages=[
        {"role": "user", "content": "北京今天天气怎么样？"}
    ],
    tools=tools,
    tool_choice="auto"  # 或 "required", "none"
)

print(response.choices[0].message)
```

### tool_choice 选项说明

| 值 | 说明 |
|---|---|
| `"auto"` | 模型自动决定是否调用工具 |
| `"required"` | 强制模型必须调用工具（vLLM >= 0.8.3） |
| `"none"` | 禁止模型调用工具 |
| `{"type": "function", "function": {"name": "xxx"}}` | 强制调用指定函数 |

---

## 常见问题排查

### 1. 工具调用不生效

**症状**：模型忽略工具，返回普通文本

**解决方案**：
```bash
# 尝试使用 tool_choice="required" 强制调用
# 检查是否使用了正确的 parser
# GLM-4.7 使用 glm47，Qwen3 使用 hermes

# Qwen3 官方推荐使用 Qwen-Agent 获得最佳工具调用体验
pip install qwen-agent
```

### 2. 工具调用解析失败

**症状**：`tool_calls` 数组为空，但内容在 `content` 字段

**解决方案**：
```bash
# 确认 vLLM 版本 >= 0.9.0
pip install vllm --upgrade

# 尝试使用自定义 chat-template
vllm serve MODEL --chat-template /path/to/template.jinja
```

### 3. UFW 导致连接超时

**症状**：API 请求间歇性超时

**解决方案**：
```bash
# 检查 UFW 日志
sudo tail -f /var/log/ufw.log

# 关闭日志
sudo ufw logging off

# 增加 conntrack 限制
sudo sysctl -w net.netfilter.nf_conntrack_max=1048576
```

### 4. 显存不足 (OOM)

**解决方案**：
```bash
# 降低 GPU 利用率
--gpu-memory-utilization 0.8

# 减少最大上下文长度
--max-model-len 16384

# 使用量化版本
Qwen/Qwen3-235B-A22B-Instruct-2507-FP8
zai-org/GLM-4.7-FP8

# 启用 expert parallel (MoE)
--enable-expert-parallel
```

### 5. MoE 模型 tensor-parallel 问题

**症状**：FP8 模型使用高 tensor-parallel 时出错

**解决方案**：
```bash
# 降低 tensor parallel 度数
--tensor-parallel-size 4

# 或启用 expert parallel
--tensor-parallel-size 8 --enable-expert-parallel
```

### 6. 推理速度慢

**解决方案**：
```bash
# GLM-4.7 启用 MTP 加速
--speculative-config.method mtp
--speculative-config.num_speculative_tokens 1

# 禁用请求日志
--disable-log-requests

# 增加批处理大小
--max-num-seqs 128
```

---

## Systemd 服务配置

### GLM-4.7 服务文件

```ini
# /etc/systemd/system/vllm-glm47.service
[Unit]
Description=vLLM GLM-4.7 API Server
After=network.target

[Service]
Type=simple
User=root
Environment="CUDA_VISIBLE_DEVICES=0,1,2,3"
ExecStart=/usr/local/bin/vllm serve zai-org/GLM-4.7-FP8 \
    --host 0.0.0.0 \
    --port 8000 \
    --tensor-parallel-size 4 \
    --enable-auto-tool-choice \
    --tool-call-parser glm47 \
    --reasoning-parser glm45 \
    --speculative-config.method mtp \
    --speculative-config.num_speculative_tokens 1 \
    --gpu-memory-utilization 0.9 \
    --max-model-len 32768 \
    --trust-remote-code \
    --disable-log-requests \
    --served-model-name glm-4.7
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

### Qwen3-235B-A22B 服务文件

```ini
# /etc/systemd/system/vllm-qwen3-235b.service
[Unit]
Description=vLLM Qwen3-235B-A22B API Server
After=network.target

[Service]
Type=simple
User=root
Environment="CUDA_VISIBLE_DEVICES=0,1,2,3,4,5,6,7"
ExecStart=/usr/local/bin/vllm serve Qwen/Qwen3-235B-A22B-Instruct-2507 \
    --host 0.0.0.0 \
    --port 8001 \
    --tensor-parallel-size 8 \
    --enable-auto-tool-choice \
    --tool-call-parser hermes \
    --reasoning-parser deepseek_r1 \
    --gpu-memory-utilization 0.9 \
    --max-model-len 32768 \
    --trust-remote-code \
    --disable-log-requests \
    --served-model-name qwen3-235b
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

### 启用服务

```bash
sudo systemctl daemon-reload
sudo systemctl enable vllm-glm47
sudo systemctl enable vllm-qwen3-235b
sudo systemctl start vllm-glm47
sudo systemctl start vllm-qwen3-235b

# 查看状态
sudo systemctl status vllm-glm47
sudo systemctl status vllm-qwen3-235b
sudo journalctl -u vllm-qwen3-235b -f
```

---

## 快速参考

### GLM-4.7 一键启动

```bash
vllm serve zai-org/GLM-4.7-FP8 --tensor-parallel-size 4 --enable-auto-tool-choice --tool-call-parser glm47 --reasoning-parser glm45 --speculative-config.method mtp --speculative-config.num_speculative_tokens 1
```

### Qwen3-235B-A22B 一键启动

```bash
vllm serve Qwen/Qwen3-235B-A22B-Instruct-2507 --tensor-parallel-size 8 --enable-auto-tool-choice --tool-call-parser hermes --reasoning-parser deepseek_r1
```

---

## 全量版本工具调用/MCP 最简配置

> 以下为**全量模型（非量化）** 部署时，启用工具调用和 MCP 支持的**最小必要参数**。

### GLM-4.7 全量版本

```bash
vllm serve zai-org/GLM-4.7 \
    --host 0.0.0.0 \
    --port 8000 \
    --tensor-parallel-size 8 \
    --enable-auto-tool-choice \
    --tool-call-parser glm47 \
    --trust-remote-code \
    --served-model-name glm-4.7
```

**必要参数说明**：

| 参数 | 值 | 说明 |
|------|-----|------|
| `--enable-auto-tool-choice` | (无值) | **必需**，启用工具自动调用 |
| `--tool-call-parser` | `glm47` | **必需**，GLM-4.7 专用解析器 |
| `--trust-remote-code` | (无值) | **必需**，GLM 模型需要 |

**可选增强参数**：

| 参数 | 值 | 说明 |
|------|-----|------|
| `--reasoning-parser` | `glm45` | 启用推理/思考解析 |
| `--speculative-config.method` | `mtp` | MTP 推测解码加速 |
| `--speculative-config.num_speculative_tokens` | `1` | 推测 token 数 |

**完整推荐命令**：

```bash
vllm serve zai-org/GLM-4.7 \
    --host 0.0.0.0 \
    --port 8000 \
    --tensor-parallel-size 8 \
    --enable-auto-tool-choice \
    --tool-call-parser glm47 \
    --reasoning-parser glm45 \
    --gpu-memory-utilization 0.9 \
    --max-model-len 32768 \
    --trust-remote-code \
    --served-model-name glm-4.7
```

---

### Qwen3-235B-A22B 全量版本

```bash
vllm serve Qwen/Qwen3-235B-A22B \
    --host 0.0.0.0 \
    --port 8001 \
    --tensor-parallel-size 8 \
    --enable-auto-tool-choice \
    --tool-call-parser hermes \
    --trust-remote-code \
    --served-model-name qwen3-235b
```

**必要参数说明**：

| 参数 | 值 | 说明 |
|------|-----|------|
| `--enable-auto-tool-choice` | (无值) | **必需**，启用工具自动调用 |
| `--tool-call-parser` | `hermes` | **必需**，Qwen3 使用 Hermes 格式 |
| `--trust-remote-code` | (无值) | **必需**，Qwen 模型需要 |

**可选增强参数**：

| 参数 | 值 | 说明 |
|------|-----|------|
| `--reasoning-parser` | `deepseek_r1` | 启用推理/思考解析 |
| `--enable-expert-parallel` | (无值) | MoE 专家并行优化 |

**完整推荐命令**：

```bash
vllm serve Qwen/Qwen3-235B-A22B \
    --host 0.0.0.0 \
    --port 8001 \
    --tensor-parallel-size 8 \
    --enable-auto-tool-choice \
    --tool-call-parser hermes \
    --reasoning-parser deepseek_r1 \
    --gpu-memory-utilization 0.9 \
    --max-model-len 32768 \
    --trust-remote-code \
    --served-model-name qwen3-235b
```

---

### 工具调用/MCP 参数速查表

| 模型 | tool-call-parser | reasoning-parser | 其他必需 |
|------|------------------|------------------|----------|
| **GLM-4.7** | `glm47` | `glm45` | `--trust-remote-code` |
| **Qwen3-235B-A22B** | `hermes` | `deepseek_r1` | `--trust-remote-code` |

### MCP 客户端配置示例

```json
{
  "llm": {
    "provider": "openai",
    "baseUrl": "http://YOUR_SERVER_IP:8000/v1",
    "model": "glm-4.7",
    "apiKey": "not-needed"
  }
}
```

```json
{
  "llm": {
    "provider": "openai",
    "baseUrl": "http://YOUR_SERVER_IP:8001/v1",
    "model": "qwen3-235b",
    "apiKey": "not-needed"
  }
}
```

---

## 参考链接

- [vLLM Tool Calling 官方文档](https://docs.vllm.ai/en/latest/features/tool_calling/)
- [GLM-4.7 部署指南](https://docs.vllm.ai/projects/recipes/en/latest/GLM/GLM.html)
- [Qwen3-235B-A22B HuggingFace](https://huggingface.co/Qwen/Qwen3-235B-A22B)
- [Qwen3-235B-A22B-Instruct-2507](https://huggingface.co/Qwen/Qwen3-235B-A22B-Instruct-2507)
- [Qwen vLLM 部署指南](https://qwen.readthedocs.io/en/latest/deployment/vllm.html)
- [Qwen Function Calling 文档](https://qwen.readthedocs.io/en/latest/framework/function_call.html)
- [Qwen GitHub](https://github.com/QwenLM/Qwen3)
- [vLLM GitHub](https://github.com/vllm-project/vllm)
