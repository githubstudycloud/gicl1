# 自定义斜杠命令 (Skills)

## 什么是 Skills

Skills 是可复用的提示模板，通过斜杠命令 `/skill-name` 触发。它们可以：

- 封装常用工作流
- 自动化重复任务
- 标准化团队流程

## 创建 Skill

### 文件位置

```
个人 Skills:     ~/.claude/skills/skill-name/SKILL.md
项目 Skills:     .claude/skills/skill-name/SKILL.md
```

### 基本结构

```markdown
---
name: my-skill
description: 这个 Skill 做什么
---

Skill 的提示内容...
```

## Skill 示例

### 代码审查

`.claude/skills/code-review/SKILL.md`:

```markdown
---
name: code-review
description: Review code for quality and best practices
allowed-tools: Read, Grep, Glob
---

审查以下代码，检查：
- 代码清晰度和可读性
- 性能问题
- 安全漏洞
- 最佳实践

提供具体的、可操作的反馈，包含行号。
```

使用：`/code-review`

### 修复 Issue

`.claude/skills/fix-issue/SKILL.md`:

```markdown
---
name: fix-issue
description: Fix a GitHub issue by number
disable-model-invocation: true
---

修复 GitHub Issue #$ARGUMENTS:

1. 运行 `gh issue view $ARGUMENTS` 获取详情
2. 理解需求
3. 定位相关代码
4. 实现修复
5. 编写测试
6. 创建提交
```

使用：`/fix-issue 123`

### Git Commit

`.claude/skills/commit/SKILL.md`:

```markdown
---
name: commit
description: Create a well-formatted git commit
disable-model-invocation: true
allowed-tools: Bash
---

创建 git commit:

1. 运行 `git status` 查看变更
2. 运行 `git diff --staged` 查看暂存的更改
3. 分析更改并起草 commit message
4. 使用 Conventional Commits 格式
5. 创建 commit

Commit 格式:
- feat: 新功能
- fix: Bug 修复
- docs: 文档更新
- style: 格式化
- refactor: 重构
- test: 测试
- chore: 其他
```

使用：`/commit`

### 部署

`.claude/skills/deploy/SKILL.md`:

```markdown
---
name: deploy
description: Deploy to production
disable-model-invocation: true
allowed-tools: Bash
---

部署 $ARGUMENTS 到生产环境:

1. 确认当前分支是 main
2. 运行 `npm run build`
3. 运行测试 `npm test`
4. 如果测试通过，执行 `npm run deploy -- --env=$ARGUMENTS`
5. 验证部署状态
```

使用：`/deploy staging` 或 `/deploy production`

### 安全审计

`.claude/skills/security-audit/SKILL.md`:

```markdown
---
name: security-audit
description: Perform security audit on the codebase
allowed-tools: Read, Grep, Glob, Bash
---

执行安全审计:

1. 扫描依赖漏洞: `npm audit`
2. 检查硬编码的密钥和凭证
3. 查找 SQL 注入风险
4. 检查 XSS 漏洞
5. 审查认证和授权逻辑
6. 检查敏感数据处理

生成安全报告，按严重程度排序。
```

## Frontmatter 字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `name` | string | Skill 名称（用于 `/name`） |
| `description` | string | 何时使用此 Skill |
| `disable-model-invocation` | boolean | `true` 时仅手动触发 |
| `user-invocable` | boolean | `false` 时隐藏命令菜单 |
| `allowed-tools` | string | 允许的工具列表 |
| `context` | string | `fork` 在子代理中运行 |

### disable-model-invocation

```markdown
---
disable-model-invocation: true  # Claude 不会自动调用
---
```

- `true`: 只能通过 `/skill-name` 手动触发
- `false`: Claude 可能基于描述自动调用

### allowed-tools

```markdown
---
allowed-tools: Read, Grep, Glob, Bash, Edit, Write
---
```

限制 Skill 可以使用的工具。

## 参数传递

### 使用 $ARGUMENTS

```markdown
---
name: search-code
---

在代码库中搜索: $ARGUMENTS

使用 grep 和 glob 找到所有相关文件。
```

使用：`/search-code authentication`

### 多个参数

```markdown
---
name: create-component
---

创建组件:
- 名称: $ARGUMENTS[0]
- 类型: $ARGUMENTS[1]
```

使用：`/create-component Button functional`

### 环境变量

```markdown
---
name: deploy
---

会话 ID: ${CLAUDE_SESSION_ID}
工作目录: ${PWD}
```

## 添加支持文件

```
my-skill/
├── SKILL.md           # 主指令（必需）
├── reference.md       # 详细参考
├── examples.md        # 使用示例
├── templates/
│   └── component.tsx  # 模板文件
└── scripts/
    └── validate.sh    # 验证脚本
```

在 SKILL.md 中引用：

```markdown
详细说明见 [reference.md](reference.md)

使用示例见 [examples.md](examples.md)
```

## 实用 Skills 集合

### PR 创建

```markdown
---
name: pr
description: Create a pull request
disable-model-invocation: true
---

创建 Pull Request:

1. 检查当前分支和更改
2. 分析所有 commit
3. 生成 PR 标题和描述
4. 使用 `gh pr create` 创建 PR

格式:
## Summary
- 要点1
- 要点2

## Test Plan
- [ ] 测试项1
- [ ] 测试项2
```

### 日志分析

```markdown
---
name: analyze-logs
description: Analyze error logs
allowed-tools: Read, Grep, Bash
---

分析日志文件 $ARGUMENTS:

1. 统计错误类型和频率
2. 识别模式和趋势
3. 找出根本原因
4. 提供解决建议
```

### 数据库迁移

```markdown
---
name: db-migrate
description: Create database migration
disable-model-invocation: true
---

创建数据库迁移:

1. 分析请求的变更: $ARGUMENTS
2. 生成迁移文件
3. 创建回滚脚本
4. 更新类型定义
5. 运行迁移测试
```

## 管理 Skills

```bash
# 查看可用 Skills
/skills

# 创建新 Skill（交互式）
/skills new

# 编辑 Skill
/skills edit skill-name
```

## 最佳实践

1. **命名清晰** - 使用描述性的 kebab-case 名称
2. **写好描述** - 帮助 Claude 决定何时自动使用
3. **限制工具** - 使用 `allowed-tools` 限制权限
4. **提供示例** - 在 Skill 中包含输入/输出示例
5. **模块化** - 每个 Skill 做一件事
6. **测试** - 手动测试后再依赖自动调用
