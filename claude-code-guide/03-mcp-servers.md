# MCP 服务器配置

## 什么是 MCP

Model Context Protocol (MCP) 是一个开放协议，允许 Claude 连接到外部工具、数据库、API 和服务，极大扩展了 Claude 的能力。

## 配置方式

### 方式1：交互式配置

```bash
# 添加 MCP 服务器
claude mcp add

# 列出已配置的服务器
claude mcp list

# 移除服务器
claude mcp remove <server-name>
```

### 方式2：编辑配置文件

配置文件位置：
- **本地（所有项目）**: `~/.mcp.json`
- **项目级**: `.mcp.json`
- **用户级**: `~/.claude/.mcp.json`

## 配置文件格式

```json
{
  "mcpServers": {
    "server-name": {
      "type": "stdio | http | sse",
      "command": "command-to-run",
      "args": ["arg1", "arg2"],
      "env": {
        "API_KEY": "xxx"
      }
    }
  }
}
```

### 传输类型

| 类型 | 说明 | 示例 |
|------|------|------|
| `stdio` | 本地进程，通过标准输入输出通信 | 数据库、文件系统 |
| `http` | HTTP 请求响应 | REST API |
| `sse` | Server-Sent Events | 实时服务 |

## 常用 MCP 服务器配置

### GitHub

```json
{
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_xxx"
      }
    }
  }
}
```

### PostgreSQL 数据库

```json
{
  "mcpServers": {
    "postgres": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-postgres"],
      "env": {
        "DATABASE_URL": "postgresql://user:pass@localhost:5432/db"
      }
    }
  }
}
```

### Slack

```json
{
  "mcpServers": {
    "slack": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "xoxb-xxx",
        "SLACK_TEAM_ID": "T123456"
      }
    }
  }
}
```

### 文件系统

```json
{
  "mcpServers": {
    "filesystem": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-filesystem", "/path/to/allowed/dir"]
    }
  }
}
```

### Sentry 错误监控

```json
{
  "mcpServers": {
    "sentry": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-sentry"],
      "env": {
        "SENTRY_AUTH_TOKEN": "xxx",
        "SENTRY_ORG": "your-org"
      }
    }
  }
}
```

### Chrome 浏览器控制

```json
{
  "mcpServers": {
    "mcp-chrome": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-chrome"]
    }
  }
}
```

### Context7 文档查询

```json
{
  "mcpServers": {
    "context7": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-context7"]
    }
  }
}
```

### Sequential Thinking 思维链

```json
{
  "mcpServers": {
    "sequential-thinking": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-sequential-thinking"]
    }
  }
}
```

### Memory 记忆图谱

```json
{
  "mcpServers": {
    "memory": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-memory"],
      "env": {
        "MEMORY_FILE": "~/.claude/memory.json"
      }
    }
  }
}
```

### SSH 远程服务器

```json
{
  "mcpServers": {
    "ssh-server": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "mcp-ssh-server"],
      "env": {
        "SSH_HOST": "server.example.com",
        "SSH_USER": "admin",
        "SSH_KEY_PATH": "~/.ssh/id_rsa"
      }
    }
  }
}
```

### 飞书 Feishu

```json
{
  "mcpServers": {
    "feishu": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "mcp-feishu-server"],
      "env": {
        "FEISHU_APP_ID": "xxx",
        "FEISHU_APP_SECRET": "xxx"
      }
    }
  }
}
```

## 使用 MCP 资源

配置好 MCP 服务器后，可以在对话中引用资源：

```bash
# GitHub 资源
@github:repos/owner/repo/issues

# Slack 消息
@slack:conversations/C123456/messages

# 数据库表
@postgres:tables/users
```

## 完整配置示例

```json
{
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_xxx"
      }
    },
    "postgres": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-postgres"],
      "env": {
        "DATABASE_URL": "postgresql://localhost:5432/mydb"
      }
    },
    "filesystem": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-filesystem", "./data"]
    },
    "context7": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-context7"]
    }
  }
}
```

## 调试 MCP 服务器

```bash
# 启用调试模式
claude --debug "mcp"

# 查看 MCP 状态
/status

# 诊断问题
/doctor
```

## 常见问题

### 服务器无法启动

1. 检查命令是否正确
2. 确认依赖已安装
3. 验证环境变量

### 权限问题

在 `settings.json` 中添加权限：

```json
{
  "permissions": {
    "allow": [
      "mcp:github:*",
      "mcp:postgres:query"
    ]
  }
}
```
