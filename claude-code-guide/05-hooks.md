# Hooks 钩子

## 什么是 Hooks

Hooks 是在特定事件发生时自动执行的脚本或命令。它们可以：

- 自动格式化代码
- 阻止危险操作
- 发送通知
- 验证更改

## Hook 事件

| 事件 | 触发时机 | 常见用途 |
|------|---------|----------|
| `SessionStart` | 会话开始/恢复 | 初始化、注入上下文 |
| `UserPromptSubmit` | 提交提示前 | 验证、转换提示 |
| `PreToolUse` | 工具执行前 | 阻止危险操作 |
| `PostToolUse` | 工具执行后 | 自动格式化 |
| `PermissionRequest` | 权限对话出现 | 自动批准/拒绝 |
| `Notification` | 需要用户输入 | 发送通知 |
| `Stop` | Claude 停止响应 | 验证完成 |
| `SubagentStart` | 子代理启动 | 设置资源 |
| `SubagentStop` | 子代理完成 | 清理资源 |
| `PreCompact` | 上下文压缩前 | 备份信息 |
| `SessionEnd` | 会话结束 | 清理工作 |

## 配置方式

在 `.claude/settings.json` 或 `~/.claude/settings.json` 中配置：

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "正则表达式",
        "hooks": [
          {
            "type": "command",
            "command": "要执行的命令"
          }
        ]
      }
    ]
  }
}
```

## 常用 Hook 示例

### 自动格式化（保存后）

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

### 自动 Lint

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npx eslint --fix 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

### 保护敏感文件

创建脚本 `.claude/hooks/protect-files.sh`:

```bash
#!/bin/bash
INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

# 保护的文件列表
PROTECTED=(
  ".env"
  ".env.local"
  "secrets.json"
  "credentials.json"
  "package-lock.json"
  ".git/"
)

for pattern in "${PROTECTED[@]}"; do
  if [[ "$FILE_PATH" == *"$pattern"* ]]; then
    echo "Blocked: $FILE_PATH is protected" >&2
    exit 2  # 退出码 2 = 阻止操作
  fi
done

exit 0  # 退出码 0 = 允许操作
```

配置:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write|Bash",
        "hooks": [
          {
            "type": "command",
            "command": "./.claude/hooks/protect-files.sh"
          }
        ]
      }
    ]
  }
}
```

### 阻止危险命令

`.claude/hooks/block-dangerous.sh`:

```bash
#!/bin/bash
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

# 危险命令模式
DANGEROUS=(
  "rm -rf /"
  "rm -rf ~"
  "rm -rf *"
  "> /dev/sda"
  "mkfs"
  "dd if="
  ":(){:|:&};:"
)

for pattern in "${DANGEROUS[@]}"; do
  if [[ "$COMMAND" == *"$pattern"* ]]; then
    echo "Blocked: Dangerous command detected" >&2
    exit 2
  fi
done

exit 0
```

### 桌面通知

**macOS**:

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "permission_prompt",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude needs approval\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

**Linux**:

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "permission_prompt",
        "hooks": [
          {
            "type": "command",
            "command": "notify-send 'Claude Code' 'Claude needs approval'"
          }
        ]
      }
    ]
  }
}
```

**Windows (PowerShell)**:

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "permission_prompt",
        "hooks": [
          {
            "type": "command",
            "command": "powershell -Command \"[System.Windows.Forms.MessageBox]::Show('Claude needs approval')\""
          }
        ]
      }
    ]
  }
}
```

### 会话开始时注入上下文

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Current git branch:' && git branch --show-current && echo 'Recent commits:' && git log --oneline -5"
          }
        ]
      }
    ]
  }
}
```

### 验证任务完成

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "npm test 2>&1 | tail -5"
          }
        ]
      }
    ]
  }
}
```

### 基于 Prompt 的 Hook

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "检查是否所有任务都已完成。返回 {\"ok\": true/false, \"reason\": \"...\"}",
            "model": "haiku"
          }
        ]
      }
    ]
  }
}
```

## Hook 退出码

| 退出码 | 含义 | 效果 |
|--------|------|------|
| 0 | 继续 | 操作进行；stdout 加入上下文 |
| 2 | 阻止 | 取消操作；stderr 反馈给 Claude |
| 其他 | 继续 | 操作进行；stderr 被记录但不显示 |

## Matcher 语法

Matcher 是正则表达式，匹配工具名或事件详情：

```json
{
  "matcher": "Edit|Write",           // 匹配 Edit 或 Write 工具
  "matcher": "Bash",                  // 匹配 Bash 工具
  "matcher": ".*",                    // 匹配所有
  "matcher": "permission_prompt"     // 匹配权限请求
}
```

## Hook 输入数据

Hook 通过 stdin 接收 JSON 数据：

```json
{
  "session_id": "xxx",
  "tool_name": "Edit",
  "tool_input": {
    "file_path": "/path/to/file.ts",
    "old_string": "...",
    "new_string": "..."
  }
}
```

使用 `jq` 解析：

```bash
#!/bin/bash
INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path')
TOOL_NAME=$(echo "$INPUT" | jq -r '.tool_name')
```

## 完整配置示例

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo '=== Session Started ===' && date"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "./.claude/hooks/protect-files.sh"
          }
        ]
      },
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "./.claude/hooks/block-dangerous.sh"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write 2>/dev/null || true"
          }
        ]
      }
    ],
    "Notification": [
      {
        "matcher": "permission_prompt",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude needs approval\" with title \"Claude Code\"'"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo '=== Claude stopped ===' && npm test --silent 2>&1 | tail -3"
          }
        ]
      }
    ]
  }
}
```

## 调试 Hooks

```bash
# 启用调试
claude --debug "hooks"

# 查看 hooks 状态
/hooks

# 测试 hook 脚本
echo '{"tool_name":"Edit","tool_input":{"file_path":"test.ts"}}' | ./.claude/hooks/protect-files.sh
```

## 最佳实践

1. **优雅失败** - Hook 失败不应阻止正常操作
2. **快速执行** - 避免长时间运行的 hook
3. **明确反馈** - 阻止时给出清晰的错误信息
4. **测试** - 独立测试每个 hook 脚本
5. **日志** - 记录 hook 执行情况便于调试
