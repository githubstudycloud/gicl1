# 输出风格配置

## 输出风格选项

Claude Code 支持多种输出风格，通过 `outputStyle` 配置：

| 风格 | 说明 |
|------|------|
| `Concise` | 简洁，最少解释 |
| `Explanatory` | 详细解释 |
| `Formal` | 正式、专业 |
| `Casual` | 随意、友好 |

## 配置方式

### 配置文件

`~/.claude/settings.json`:

```json
{
  "outputStyle": "Concise"
}
```

### 会话中切换

```bash
/config
# 搜索 outputStyle
```

## 语言设置

### 配置响应语言

```json
{
  "language": "chinese"
}
```

可选值：
- `english`
- `chinese`
- `japanese`
- `korean`
- `spanish`
- `french`
- `german`
- 等等...

### 在 CLAUDE.md 中指定

```markdown
# 语言偏好

请始终使用中文回复。
代码注释使用英文。
Commit message 使用英文。
```

## 自定义输出格式

### 通过 CLAUDE.md

```markdown
# 输出风格指南

## 代码风格
- 不添加不必要的注释
- 变量名使用驼峰命名
- 函数要有 JSDoc 注释

## 解释风格
- 简洁明了
- 先给结论，再解释原因
- 使用列表而非长段落

## 格式要求
- Markdown 格式输出
- 代码块指定语言
- 使用表格展示对比
```

### 通过系统提示

```bash
claude --append-system-prompt "Always respond in bullet points. Keep explanations under 100 words."
```

## 代码输出控制

### 只输出代码

```markdown
# CLAUDE.md
当修改代码时：
- 不要解释你在做什么
- 直接给出代码
- 除非我问，否则不要解释
```

### 详细解释

```markdown
# CLAUDE.md
当修改代码时：
1. 先解释要修改什么
2. 说明为什么这样改
3. 给出代码
4. 说明如何测试
```

## Token 输出限制

### 配置最大输出

```json
{
  "env": {
    "CLAUDE_CODE_MAX_OUTPUT_TOKENS": "128000"
  }
}
```

### 限制响应长度

```bash
# 简短响应
claude -p "Explain this in 50 words or less"

# 详细响应
claude -p "Give me a comprehensive explanation"
```

## 输出格式选项

### 交互模式（默认）

标准 Markdown 格式输出。

### JSON 格式

```bash
claude -p "List all functions" --output-format json
```

输出：

```json
{
  "functions": [
    {"name": "getUserById", "file": "src/user.ts", "line": 10},
    {"name": "createUser", "file": "src/user.ts", "line": 25}
  ]
}
```

### 流式 JSON

```bash
claude -p "Process files" --output-format stream-json
```

实时输出每个处理结果。

### 纯文本

```bash
claude -p "Summarize" --output-format text
```

## 状态行配置

### 启用状态行

```bash
/statusline
```

### 自定义状态行

显示：
- 当前模型
- 上下文使用
- 费用统计
- 权限模式

### 配置示例

```json
{
  "statusLine": {
    "enabled": true,
    "showModel": true,
    "showContext": true,
    "showCost": true,
    "showPermissionMode": true
  }
}
```

## 日志详细程度

### 详细模式

```bash
# 启动时
claude --verbose

# 会话中
Ctrl+O  # 切换
```

### 调试模式

```bash
# 全部调试
claude --debug

# 按类别
claude --debug "api,hooks,mcp"
```

## 通知设置

### 桌面通知

通过 Hooks 配置：

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"$MESSAGE\" with title \"Claude\"'"
          }
        ]
      }
    ]
  }
}
```

### 声音提示

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "afplay /System/Library/Sounds/Glass.aiff"
          }
        ]
      }
    ]
  }
}
```

## 针对不同场景的风格

### 代码审查

```markdown
# CLAUDE.md - 代码审查风格

审查时使用以下格式：

## 问题类型标记
- 🔴 Critical: 必须修复
- 🟡 Warning: 建议修复
- 🔵 Info: 可选改进

## 每个问题包含
1. 文件和行号
2. 问题描述
3. 修复建议
4. 示例代码（如适用）
```

### 文档生成

```markdown
# CLAUDE.md - 文档风格

生成文档时：
- 使用 JSDoc/TSDoc 格式
- 包含参数说明
- 包含返回值说明
- 包含使用示例
- 包含异常说明
```

### 教学模式

```markdown
# CLAUDE.md - 教学风格

解释代码时：
1. 先给出高层概述
2. 分步骤解释
3. 使用类比帮助理解
4. 指出常见陷阱
5. 提供进一步学习资源
```

## 实用配置模板

### 极简风格

```json
{
  "outputStyle": "Concise",
  "env": {
    "CLAUDE_CODE_MAX_OUTPUT_TOKENS": "4096"
  }
}
```

CLAUDE.md:
```markdown
- 直接给代码，不解释
- 用最少的改动
- 不加注释除非必要
```

### 详细风格

```json
{
  "outputStyle": "Explanatory",
  "alwaysThinkingEnabled": true
}
```

CLAUDE.md:
```markdown
- 详细解释每个改动
- 说明原因和影响
- 提供替代方案
- 包含测试建议
```

### 专业风格

```json
{
  "outputStyle": "Formal",
  "language": "english"
}
```

CLAUDE.md:
```markdown
- Use formal technical language
- Follow enterprise coding standards
- Include comprehensive documentation
- Cite relevant best practices
```
