# 高级技巧与最佳实践

## 上下文管理（最重要！）

### 为什么重要

上下文窗口会迅速填满，影响：
- 响应速度
- 回答质量
- Token 成本

### 管理技巧

```bash
# 查看上下文使用情况
/context

# 压缩上下文（保留关键信息）
/compact "保留 auth 模块的更改，删除旧的探索内容"

# 完全清空
/clear

# 创建检查点
/checkpoint

# 回退到检查点
/rewind
```

### 最佳实践

1. **频繁压缩** - 完成子任务后压缩
2. **明确指令** - 告诉 Claude 保留什么
3. **分会话** - 不同任务用不同会话
4. **使用子代理** - 探索任务委托给子代理

## 编写有效的 CLAUDE.md

### 应该包含

```markdown
# 项目说明

## 技术栈
- Next.js 14 + TypeScript
- Tailwind CSS
- PostgreSQL + Prisma

## 常用命令
- `pnpm dev` - 开发服务器
- `pnpm test` - 运行测试
- `pnpm db:migrate` - 数据库迁移

## 代码风格
- 使用 ES 模块
- 优先使用函数组件
- 错误处理使用 try-catch

## 项目特殊性
- API 路由在 /app/api/
- 环境变量在 .env.local
- 不要修改 package-lock.json
```

### 不应该包含

- 标准语言约定
- 可以从代码推断的内容
- 频繁变化的信息

## 子代理 (Subagents)

### 创建子代理

`.claude/agents/security-reviewer.md`:

```markdown
---
name: security-reviewer
description: Security audit specialist
tools: Read, Grep, Glob
model: opus
permissionMode: plan
---

你是安全审计专家。检查：
- SQL 注入
- XSS 漏洞
- 认证缺陷
- 敏感数据暴露
- 依赖漏洞

输出按严重程度排序的报告。
```

### 使用子代理

```bash
# 自动委托
Use a subagent to review security

# 显式调用
Use the security-reviewer subagent to audit the auth module
```

### 子代理最佳实践

1. **单一职责** - 每个子代理做一件事
2. **限制工具** - 只给必要的权限
3. **选择模型** - 复杂任务用 opus，简单用 haiku
4. **隔离上下文** - 子代理不会污染主会话

## 并行工作流

### Git Worktrees

```bash
# 创建工作树
git worktree add ../feature-auth -b feature/authentication

# 在不同工作树运行 Claude
cd ../feature-auth
claude

# 完成后清理
git worktree remove ../feature-auth
```

### 多窗口/会话

```bash
# 终端 1: 前端开发
cd frontend && claude

# 终端 2: 后端开发
cd backend && claude

# 终端 3: 测试
cd tests && claude
```

## 自动化工作流

### CI/CD 集成

```bash
# 代码审查
claude -p "Review this PR for issues" < <(git diff main...HEAD)

# 自动生成 changelog
claude -p "Generate changelog" --output-format json > changelog.json

# 安全扫描
claude -p "Security audit" --max-turns 10
```

### 批量处理

```bash
# 迁移多个文件
for file in src/**/*.js; do
  claude -p "Migrate $file to TypeScript" --max-turns 3
done
```

### 定时任务

```bash
# crontab 每日审查
0 9 * * * cd /project && claude -p "/daily-review" >> /var/log/claude-review.log
```

## 提示词技巧

### 结构化请求

```
任务：实现用户认证
要求：
1. 使用 JWT
2. 支持刷新 token
3. 包含登录/登出/注册端点

约束：
- 不修改现有数据库结构
- 使用现有的 User 模型
- 错误消息要友好

输出：
- 实现代码
- 单元测试
- API 文档
```

### 分步执行

```
第一步：分析现有代码结构
第二步：设计实现方案（不要写代码）
第三步：等我确认后再实现
```

### 指定格式

```
以 JSON 格式输出：
{
  "files_changed": [...],
  "tests_added": [...],
  "breaking_changes": [...]
}
```

## 调试技巧

### 启用调试模式

```bash
# 全部调试信息
claude --debug

# 按类别过滤
claude --debug "api,hooks,mcp"

# 详细日志
claude --verbose
```

### 诊断问题

```bash
# 运行诊断
/doctor

# 查看状态
/status

# 检查 hooks
/hooks
```

### 常见问题

**上下文超限**
```bash
/compact "保留最重要的内容"
```

**API 超时**
```bash
export ANTHROPIC_TIMEOUT=120000
```

**权限问题**
```json
{
  "permissions": {
    "allow": ["Bash(npm *)"]
  }
}
```

## 安全最佳实践

### 保护敏感文件

```json
{
  "permissions": {
    "deny": [
      "Read(.env*)",
      "Read(**/secrets/**)",
      "Read(**/*credential*)",
      "Edit(.git/**)"
    ]
  }
}
```

### 限制命令

```json
{
  "permissions": {
    "deny": [
      "Bash(rm -rf *)",
      "Bash(curl * | bash)",
      "Bash(wget * | sh)",
      "Bash(* > /dev/*)"
    ]
  }
}
```

### 审计 Hook

```bash
#!/bin/bash
# .claude/hooks/audit.sh
INPUT=$(cat)
echo "[$(date)] $INPUT" >> ~/.claude/audit.log
exit 0
```

## 团队协作

### 共享配置

```
项目根目录/
├── .claude/
│   ├── settings.json      # 团队配置（提交）
│   ├── settings.local.json # 个人配置（gitignore）
│   ├── CLAUDE.md          # 项目说明（提交）
│   └── skills/            # 共享 Skills（提交）
```

### 标准化 Skills

创建团队通用的 Skills：

```markdown
---
name: team-pr
description: Create PR following team standards
---

创建 PR 遵循团队规范：
- 标题格式: [类型] 简短描述
- 描述包含: 背景、变更、测试计划
- 链接相关 Issue
- 请求至少两位审查者
```

### 代码风格一致性

使用 PostToolUse Hook 自动格式化：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write $(jq -r '.tool_input.file_path')"
          }
        ]
      }
    ]
  }
}
```

## 性能优化

### 减少延迟

1. 使用 haiku 处理简单任务
2. 限制扩展思考 token
3. 使用本地缓存的 MCP 服务器

### 减少成本

1. 频繁压缩上下文
2. 使用子代理隔离探索任务
3. 批量操作时使用 `--max-turns` 限制

### 提高准确性

1. 复杂任务使用 opus
2. 启用扩展思考
3. 提供清晰的 CLAUDE.md
4. 使用 Plan 模式先规划

## 实用工作流示例

### 功能开发流程

```bash
# 1. 规划
claude --permission-mode plan
> 设计用户通知系统

# 2. 创建计划文件
Ctrl+G  # 编辑计划

# 3. 执行
Shift+Tab  # 切回 normal 模式
> 按计划实现

# 4. 测试
> 编写并运行测试

# 5. 提交
/commit
```

### Bug 修复流程

```bash
# 1. 分析
claude --model opus
> 分析这个错误: [粘贴错误]

# 2. 定位
> 找到相关代码

# 3. 修复
/model sonnet
> 修复这个 bug

# 4. 验证
> 运行相关测试
```

### 代码审查流程

```bash
# 1. 获取变更
> Review changes since last commit

# 2. 详细审查
Use the code-reviewer subagent to analyze security

# 3. 生成报告
> Generate review report in markdown
```
