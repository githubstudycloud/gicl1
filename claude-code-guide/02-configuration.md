# 配置详解

## 配置文件位置与优先级

配置按以下优先级加载（高到低）：

1. **Managed settings** - 系统级，企业 IT 部署
2. **命令行参数** - 启动时指定
3. **Local settings** - `.claude/settings.local.json`（不提交到 git）
4. **Project settings** - `.claude/settings.json`（提交到 git）
5. **User settings** - `~/.claude/settings.json`（全局）

## 完整配置示例

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",

  "permissions": {
    "allow": [
      "Bash(npm run *)",
      "Bash(git *)",
      "Bash(pnpm *)",
      "Read(./src/**)",
      "Edit(./src/**)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Read(.env)",
      "Read(./secrets/**)"
    ],
    "defaultMode": "acceptEdits"
  },

  "model": "sonnet",

  "env": {
    "ANTHROPIC_API_KEY": "sk-...",
    "ANTHROPIC_BASE_URL": "https://api.anthropic.com",
    "CLAUDE_CODE_MAX_OUTPUT_TOKENS": "128000"
  },

  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true
  },

  "alwaysThinkingEnabled": true,
  "outputStyle": "Explanatory",
  "language": "chinese",
  "respectGitignore": true
}
```

## 权限配置详解

### 权限规则语法

```
工具名(参数模式)
```

### 常用权限规则

```json
{
  "permissions": {
    "allow": [
      "Bash(npm *)",           // 允许所有 npm 命令
      "Bash(git status)",      // 只允许 git status
      "Bash(git diff *)",      // 允许 git diff
      "Read(./src/**)",        // 允许读取 src 目录
      "Edit(./src/**/*.ts)",   // 允许编辑 ts 文件
      "Write(./src/**)"        // 允许创建文件
    ],
    "deny": [
      "Bash(rm -rf *)",        // 禁止危险删除
      "Bash(curl * | *)",      // 禁止危险管道
      "Read(.env*)",           // 禁止读取环境文件
      "Read(**/*secret*)",     // 禁止读取密钥文件
      "WebFetch(*)"            // 禁止网络请求
    ]
  }
}
```

### 权限模式

```json
{
  "permissions": {
    "defaultMode": "normal"  // normal | acceptEdits | plan | dontAsk | bypassPermissions
  }
}
```

## 模型配置

### 模型别名

| 别名 | 说明 |
|------|------|
| `default` | 推荐默认模型 |
| `opus` | Claude Opus (最强) |
| `sonnet` | Claude Sonnet (平衡) |
| `haiku` | Claude Haiku (最快) |

### 配置示例

```json
{
  "model": "sonnet",
  "env": {
    "ANTHROPIC_MODEL": "claude-sonnet-4-20250514",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "claude-opus-4-5-20251101",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "claude-sonnet-4-20250514",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "claude-haiku-3-5-20241022",
    "CLAUDE_CODE_SUBAGENT_MODEL": "haiku"
  }
}
```

## 环境变量配置

### API 相关

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "sk-...",
    "ANTHROPIC_BASE_URL": "https://api.example.com",
    "ANTHROPIC_MODEL": "claude-sonnet-4-20250514"
  }
}
```

### 代理配置

```json
{
  "env": {
    "HTTP_PROXY": "http://proxy:8080",
    "HTTPS_PROXY": "http://proxy:8080",
    "NO_PROXY": "localhost,127.0.0.1"
  }
}
```

### 性能配置

```json
{
  "env": {
    "CLAUDE_CODE_MAX_OUTPUT_TOKENS": "128000",
    "MAX_THINKING_TOKENS": "50000",
    "CLAUDE_CODE_EFFORT_LEVEL": "medium"
  }
}
```

### 功能开关

```json
{
  "env": {
    "DISABLE_TELEMETRY": "1",
    "DISABLE_AUTOUPDATER": "1",
    "DISABLE_PROMPT_CACHING": "0"
  }
}
```

## 沙箱配置

```json
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true,
    "allowedPaths": [
      "/tmp",
      "./node_modules"
    ]
  }
}
```

## 项目本地配置

创建 `.claude/settings.local.json`（自动被 gitignore）：

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "my-personal-key"
  },
  "permissions": {
    "defaultMode": "acceptEdits"
  }
}
```

## CLAUDE.md 项目说明文件

在项目根目录创建 `CLAUDE.md` 或 `.claude/CLAUDE.md`：

```markdown
# 项目说明

## 代码风格
- 使用 ES 模块 (import/export)
- 2 空格缩进
- 使用 TypeScript

## 常用命令
- `pnpm dev` - 启动开发服务器
- `pnpm test` - 运行测试
- `pnpm build` - 构建项目

## 项目结构
- src/components/ - React 组件
- src/utils/ - 工具函数
- src/api/ - API 接口

## 注意事项
- 修改后运行 `pnpm type-check`
- 提交前运行 lint
- 不要修改 .env 文件
```

### CLAUDE.md 位置优先级

1. 企业级: `/Library/Application Support/ClaudeCode/CLAUDE.md` (macOS)
2. 项目级: `.claude/CLAUDE.md` (提交到 git)
3. 项目本地: `CLAUDE.local.md` (不提交)
4. 用户级: `~/.claude/CLAUDE.md` (全局)

## 配置命令

```bash
# 打开配置菜单
/config

# 查看当前状态
/status

# 编辑记忆文件
/memory

# 诊断问题
/doctor
```
