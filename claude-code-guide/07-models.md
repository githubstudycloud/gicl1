# 模型与扩展思考

## 模型选择

### 可用模型

| 模型别名 | 全称 | 特点 | 最佳用途 |
|----------|------|------|----------|
| `opus` | Claude Opus 4.5 | 最强推理能力 | 复杂架构、难题分析 |
| `sonnet` | Claude Sonnet 4 | 平衡性能与速度 | 日常编码任务 |
| `haiku` | Claude Haiku 3.5 | 最快响应 | 简单任务、后台处理 |

### 切换模型

**会话中切换**

```bash
/model
# 使用方向键选择模型
```

**启动时指定**

```bash
claude --model opus
claude --model sonnet
claude --model haiku
```

**配置文件**

```json
{
  "model": "sonnet"
}
```

**环境变量**

```bash
export ANTHROPIC_MODEL=claude-sonnet-4-20250514
```

### 模型选择策略

| 场景 | 推荐模型 |
|------|----------|
| 复杂架构设计 | opus |
| 重构大型代码库 | opus |
| 日常编码 | sonnet |
| 简单修复 | haiku |
| 代码审查 | sonnet |
| 快速问答 | haiku |
| 文档生成 | sonnet |

### 子代理模型

配置子代理使用不同模型：

```json
{
  "env": {
    "CLAUDE_CODE_SUBAGENT_MODEL": "haiku"
  }
}
```

## 扩展思考 (Extended Thinking)

### 什么是扩展思考

扩展思考让 Claude 在回答前进行更深入的推理，适用于：

- 复杂问题分析
- 多步骤规划
- 架构决策
- 疑难 bug 排查

### 启用/禁用

**快捷键切换**

```
Option+T (Mac) / Alt+T (Windows/Linux)
```

**配置默认启用**

```json
{
  "alwaysThinkingEnabled": true
}
```

**命令行**

```bash
/config
# 搜索 "thinking" 并启用
```

### 限制思考 Token

```bash
export MAX_THINKING_TOKENS=50000
```

### 何时使用

**适合扩展思考**：
- 复杂架构决策
- 多文件重构
- 性能优化分析
- 安全审计
- 难以重现的 bug

**不需要扩展思考**：
- 简单代码生成
- 格式化
- 简单问答
- 文件操作

## 努力级别 (Effort Level)

仅 Opus 模型支持自适应推理努力级别。

### 配置

```bash
# 环境变量
export CLAUDE_CODE_EFFORT_LEVEL=medium  # low | medium | high
```

**在 /model 中调整**

```bash
/model
# 选择 opus 后，使用 ← → 键调整努力级别
```

### 级别说明

| 级别 | 说明 | 用途 |
|------|------|------|
| low | 快速响应 | 简单任务 |
| medium | 平衡 | 一般任务 |
| high | 深度思考 | 复杂任务 |

## 混合模型策略

### opusplan 模式

使用 Opus 规划，Sonnet 执行：

```bash
claude --model opusplan
```

配置：

```json
{
  "model": "opusplan"
}
```

### 手动混合

1. 使用 Opus 进入 Plan 模式设计方案
2. 切换到 Sonnet 执行实现
3. 用 Haiku 处理简单后续任务

```bash
# 1. 规划
claude --model opus --permission-mode plan
> 设计认证系统架构

# 2. 执行
/model sonnet
> 按计划实现认证系统

# 3. 后续
/model haiku
> 添加注释
```

## 长上下文支持

### 100 万 Token 上下文

```bash
claude --model sonnet[1m]
```

适用于：
- 分析大型代码库
- 长对话会话
- 大量文档处理

### 上下文管理

```bash
# 查看上下文使用情况
/context

# 压缩上下文
/compact "保留最近的更改"

# 清空上下文
/clear
```

## 成本优化

### Token 消耗

| 操作 | Token 消耗 |
|------|------------|
| 读取大文件 | 高 |
| 扩展思考 | 高 |
| 简单对话 | 低 |
| 子代理任务 | 中 |

### 优化策略

1. **选择合适模型**
   ```bash
   # 简单任务用 haiku
   claude --model haiku -p "格式化这个 JSON"
   ```

2. **限制思考 Token**
   ```bash
   export MAX_THINKING_TOKENS=10000
   ```

3. **积极压缩上下文**
   ```bash
   /compact "只保留 API 相关内容"
   ```

4. **使用子代理处理大任务**
   ```bash
   Use a subagent to explore the codebase
   ```

5. **检查费用**
   ```bash
   /cost
   ```

### 费用统计

```bash
# 查看当前会话费用
/cost

# 查看状态（包含费用信息）
/status
```

## 自定义模型端点

### 使用第三方 API

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "your-api-key",
    "ANTHROPIC_BASE_URL": "https://your-api-endpoint.com",
    "ANTHROPIC_MODEL": "custom-model-name"
  }
}
```

### 使用代理

```json
{
  "env": {
    "HTTPS_PROXY": "http://proxy:8080",
    "HTTP_PROXY": "http://proxy:8080"
  }
}
```

## 模型相关命令

```bash
# 切换模型
/model

# 查看当前模型
/status

# 配置默认值
/config
```
