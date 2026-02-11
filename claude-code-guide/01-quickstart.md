# 快速开始

## 安装方法

### macOS

```bash
# 推荐：使用 Homebrew
brew install claude

# 或使用安装脚本
curl https://download.claude.com/install.sh | bash
```

### Linux

```bash
# 使用安装脚本
curl https://download.claude.com/install.sh | bash

# 或使用 npm（已弃用但仍可用）
npm install -g @anthropic-ai/claude-code
```

### Windows

```bash
# 推荐：使用 winget
winget install Anthropic.ClaudeCode

# 或使用 npm
npm install -g @anthropic-ai/claude-code
```

> **注意**: Windows 用户需要安装 Git 和 Git Bash

### 验证安装

```bash
claude --version
```

## 身份验证

首次运行时会自动打开浏览器进行登录：

```bash
claude
# 浏览器将自动打开进行 OAuth 认证
```

### 使用 API Key

如果使用自托管或第三方 API：

```bash
export ANTHROPIC_API_KEY="your-api-key"
export ANTHROPIC_BASE_URL="https://your-api-endpoint.com"
```

## 基本使用

### 启动交互会话

```bash
# 直接启动
claude

# 在特定目录启动
claude --add-dir /path/to/project

# 带初始提示启动
claude "帮我分析这个项目的结构"
```

### 会话管理

```bash
# 继续最近的对话
claude -c
claude --continue

# 选择历史会话恢复
claude -r
claude --resume

# 恢复指定会话
claude --resume "session-name"

# 从 PR 恢复相关会话
claude --from-pr 123
```

### 常用会话内命令

| 命令 | 功能 |
|------|------|
| `/help` | 显示帮助 |
| `/clear` | 清空上下文 |
| `/model` | 切换模型 |
| `/config` | 打开配置 |
| `/status` | 显示状态 |
| `/cost` | 显示费用统计 |
| `/compact` | 压缩上下文 |
| `/exit` 或 `Ctrl+D` | 退出 |

## 权限模式

Claude Code 有多种权限模式，用 `Shift+Tab` 循环切换：

| 模式 | 说明 |
|------|------|
| `normal` | 默认，每个操作都询问 |
| `acceptEdits` | 自动批准文件编辑 |
| `plan` | 只读模式，仅探索不修改 |
| `dontAsk` | 自动拒绝未预批准的操作 |
| `bypassPermissions` | 跳过所有检查（危险） |

```bash
# 启动时指定模式
claude --permission-mode plan
```

## 文件引用

使用 `@` 符号引用文件或目录：

```bash
# 引用文件
@src/main.ts 解释这个文件

# 引用目录
@src/ 列出所有组件

# 引用特定行
@src/main.ts:10-50 优化这段代码

# 引用 URL
@https://example.com/api 分析这个 API
```

## 输出模式

### 交互模式（默认）

```bash
claude
```

### 打印模式（非交互）

```bash
# 单次查询
claude -p "解释什么是 REST API"

# 从标准输入读取
cat error.log | claude -p "分析这个错误"

# JSON 输出
claude -p "列出所有函数" --output-format json

# 流式 JSON
claude -p "处理数据" --output-format stream-json
```

## 下一步

- [配置详解](02-configuration.md) - 学习如何配置 Claude Code
- [MCP 服务器](03-mcp-servers.md) - 扩展 Claude 的能力
