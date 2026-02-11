# Claude Code 速查表

## 安装

```bash
# macOS
brew install claude

# Linux
curl https://download.claude.com/install.sh | bash

# Windows
winget install Anthropic.ClaudeCode

# 验证
claude --version
```

## 启动选项

```bash
claude                          # 交互会话
claude "prompt"                 # 带初始提示
claude -c                       # 继续上次对话
claude -r                       # 选择历史会话
claude --model opus             # 指定模型
claude --permission-mode plan   # 计划模式
claude -p "query"               # 打印模式（非交互）
claude -p "query" --output-format json  # JSON 输出
```

## 会话内命令

| 命令 | 功能 |
|------|------|
| `/help` | 帮助 |
| `/model` | 切换模型 |
| `/config` | 配置 |
| `/clear` | 清空上下文 |
| `/compact` | 压缩上下文 |
| `/context` | 上下文统计 |
| `/cost` | 费用统计 |
| `/status` | 状态信息 |
| `/doctor` | 诊断 |
| `/memory` | 编辑记忆 |
| `/hooks` | 管理钩子 |
| `/skills` | 管理技能 |
| `/agents` | 管理代理 |
| `/rewind` | 回退检查点 |

## 快捷键

| 快捷键 | 功能 |
|--------|------|
| `Ctrl+C` | 中断 |
| `Ctrl+D` | 退出 |
| `Ctrl+G` | 外部编辑器 |
| `Ctrl+R` | 搜索历史 |
| `Shift+Tab` | 循环权限模式 |
| `Option/Alt+T` | 切换扩展思考 |
| `Cmd/Meta+P` | 模型选择器 |

## 文件引用

```bash
@file.ts              # 引用文件
@src/                 # 引用目录
@file.ts:10-50        # 引用行范围
@https://url.com      # 引用 URL
```

## 权限模式

| 模式 | 说明 |
|------|------|
| `normal` | 每个操作询问 |
| `acceptEdits` | 自动批准编辑 |
| `plan` | 只读模式 |
| `dontAsk` | 自动拒绝 |
| `bypassPermissions` | 跳过检查 |

## 模型选择

| 别名 | 用途 |
|------|------|
| `opus` | 复杂任务 |
| `sonnet` | 日常编码 |
| `haiku` | 简单任务 |

## 配置文件位置

```
~/.claude/settings.json           # 用户全局
.claude/settings.json             # 项目（提交）
.claude/settings.local.json       # 项目本地
~/.mcp.json                       # MCP 全局
.mcp.json                         # MCP 项目
~/.claude/CLAUDE.md               # 说明全局
.claude/CLAUDE.md                 # 说明项目
```

## 基础配置模板

```json
{
  "model": "sonnet",
  "permissions": {
    "allow": ["Bash(npm *)", "Bash(git *)"],
    "deny": ["Bash(rm -rf *)", "Read(.env*)"],
    "defaultMode": "acceptEdits"
  },
  "env": {
    "CLAUDE_CODE_MAX_OUTPUT_TOKENS": "128000"
  }
}
```

## MCP 服务器配置

```json
{
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-github"],
      "env": { "GITHUB_TOKEN": "ghp_xxx" }
    }
  }
}
```

## Skill 模板

```markdown
---
name: my-skill
description: 描述
allowed-tools: Read, Edit, Bash
---

Skill 内容...
$ARGUMENTS 获取参数
```

## Hook 配置

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{
        "type": "command",
        "command": "npx prettier --write $(jq -r '.tool_input.file_path')"
      }]
    }]
  }
}
```

## 环境变量

```bash
# API
ANTHROPIC_API_KEY=sk-xxx
ANTHROPIC_BASE_URL=https://api.anthropic.com
ANTHROPIC_MODEL=claude-sonnet-4-20250514

# 性能
CLAUDE_CODE_MAX_OUTPUT_TOKENS=128000
MAX_THINKING_TOKENS=50000

# 功能
DISABLE_TELEMETRY=1
DISABLE_AUTOUPDATER=1
```

## CLI 标志

```bash
--model <model>           # 模型
--permission-mode <mode>  # 权限模式
--tools "Tool1,Tool2"     # 限制工具
--add-dir <dir>           # 添加目录
--verbose                 # 详细输出
--debug                   # 调试模式
--output-format json      # JSON 输出
--max-turns <n>           # 限制轮数
```

## 常用工作流

### 代码审查
```bash
claude --permission-mode plan
> Review @src/ for issues
```

### 修复 Bug
```bash
claude --model opus
> 分析并修复: [错误信息]
```

### 创建 Commit
```bash
/commit
# 或使用 Skill
```

### 创建 PR
```bash
> Create a PR for current changes
```

## 调试

```bash
claude --debug            # 调试模式
/doctor                   # 诊断
/status                   # 状态
/context                  # 上下文
```

## 有用链接

- 官方文档: https://docs.anthropic.com/claude-code
- GitHub: https://github.com/anthropics/claude-code
- 问题反馈: https://github.com/anthropics/claude-code/issues
