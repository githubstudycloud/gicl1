# vLLM 全量模型工具调用部署速查

> 专注于 **GLM-4.7** 和 **Qwen3-235B-A22B** 全量版本的工具调用/MCP 部署

---

## 一、GLM-4.7 全量部署

### 最简启动命令

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

### 推荐生产命令

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
    --disable-log-requests \
    --served-model-name glm-4.7
```

### 必要参数

| 参数 | 值 | 说明 |
|------|-----|------|
| `--enable-auto-tool-choice` | - | **必需**，启用工具调用 |
| `--tool-call-parser` | `glm47` | **必需**，GLM-4.7 解析器 |
| `--trust-remote-code` | - | **必需** |

### 可选参数

| 参数 | 值 | 说明 |
|------|-----|------|
| `--reasoning-parser` | `glm45` | 启用思考/推理解析 |
| `--speculative-config.method` | `mtp` | MTP 加速 |

---

## 二、Qwen3-235B-A22B 全量部署

> MoE 架构：总参数 235B，激活参数 22B

### 最简启动命令

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

### 推荐生产命令

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
    --disable-log-requests \
    --served-model-name qwen3-235b
```

### 必要参数

| 参数 | 值 | 说明 |
|------|-----|------|
| `--enable-auto-tool-choice` | - | **必需**，启用工具调用 |
| `--tool-call-parser` | `hermes` | **必需**，Qwen3 解析器 |
| `--trust-remote-code` | - | **必需** |

### 可选参数

| 参数 | 值 | 说明 |
|------|-----|------|
| `--reasoning-parser` | `deepseek_r1` | 启用思考/推理解析 |
| `--enable-expert-parallel` | - | MoE 专家并行优化 |

---

## 三、参数速查表

| 模型 | tool-call-parser | reasoning-parser | tensor-parallel |
|------|------------------|------------------|-----------------|
| GLM-4.7 | `glm47` | `glm45` | 8 |
| Qwen3-235B-A22B | `hermes` | `deepseek_r1` | 8 |

---

## 四、MCP 客户端配置

### GLM-4.7

```json
{
  "llm": {
    "provider": "openai",
    "baseUrl": "http://YOUR_SERVER:8000/v1",
    "model": "glm-4.7",
    "apiKey": "not-needed"
  }
}
```

### Qwen3-235B-A22B

```json
{
  "llm": {
    "provider": "openai",
    "baseUrl": "http://YOUR_SERVER:8001/v1",
    "model": "qwen3-235b",
    "apiKey": "not-needed"
  }
}
```

---

## 五、工具调用测试

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://YOUR_SERVER:8000/v1",  # 或 8001
    api_key="not-needed"
)

tools = [{
    "type": "function",
    "function": {
        "name": "get_weather",
        "description": "获取天气",
        "parameters": {
            "type": "object",
            "properties": {
                "city": {"type": "string", "description": "城市"}
            },
            "required": ["city"]
        }
    }
}]

response = client.chat.completions.create(
    model="glm-4.7",  # 或 qwen3-235b
    messages=[{"role": "user", "content": "北京天气如何？"}],
    tools=tools,
    tool_choice="auto"
)

print(response.choices[0].message)
```

---

## 六、UFW 防火墙配置

### 对特定 IP 开放端口（无速率限制）

```bash
# 对单个 IP 开放
sudo ufw allow from 192.168.1.100 to any port 8000 proto tcp
sudo ufw allow from 192.168.1.100 to any port 8001 proto tcp

# 对子网开放
sudo ufw allow from 192.168.1.0/24 to any port 8000 proto tcp
sudo ufw allow from 192.168.1.0/24 to any port 8001 proto tcp
```

### 关闭日志（减少性能开销）

```bash
sudo ufw logging off
```

### 优化高并发连接

```bash
# 添加到 /etc/sysctl.conf
net.netfilter.nf_conntrack_max = 1048576
net.core.somaxconn = 65535

# 应用
sudo sysctl -p
```

---

## 七、Systemd 服务

### GLM-4.7

```ini
# /etc/systemd/system/vllm-glm47.service
[Unit]
Description=vLLM GLM-4.7
After=network.target

[Service]
Type=simple
Environment="CUDA_VISIBLE_DEVICES=0,1,2,3,4,5,6,7"
ExecStart=/usr/local/bin/vllm serve zai-org/GLM-4.7 \
    --host 0.0.0.0 --port 8000 \
    --tensor-parallel-size 8 \
    --enable-auto-tool-choice \
    --tool-call-parser glm47 \
    --reasoning-parser glm45 \
    --trust-remote-code \
    --served-model-name glm-4.7
Restart=always

[Install]
WantedBy=multi-user.target
```

### Qwen3-235B-A22B

```ini
# /etc/systemd/system/vllm-qwen3.service
[Unit]
Description=vLLM Qwen3-235B-A22B
After=network.target

[Service]
Type=simple
Environment="CUDA_VISIBLE_DEVICES=0,1,2,3,4,5,6,7"
ExecStart=/usr/local/bin/vllm serve Qwen/Qwen3-235B-A22B \
    --host 0.0.0.0 --port 8001 \
    --tensor-parallel-size 8 \
    --enable-auto-tool-choice \
    --tool-call-parser hermes \
    --reasoning-parser deepseek_r1 \
    --trust-remote-code \
    --served-model-name qwen3-235b
Restart=always

[Install]
WantedBy=multi-user.target
```

### 启用服务

```bash
sudo systemctl daemon-reload
sudo systemctl enable vllm-glm47 vllm-qwen3
sudo systemctl start vllm-glm47 vllm-qwen3
```
