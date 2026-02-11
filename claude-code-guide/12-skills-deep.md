# Skills 深度指南

## Skills 工作原理

Skills 是 Claude Code 的扩展机制，通过 SKILL.md 文件定义可复用的提示模板。

### 加载机制

1. **元数据扫描** - Claude 扫描所有 Skills（约 100 tokens/skill）
2. **相关性判断** - 基于 description 决定是否加载
3. **完整加载** - 激活时加载全部内容（<5k tokens）

### 跨平台兼容

2025 年 12 月，Anthropic 发布了 Agent Skills 开放标准，以下工具都支持：

- Claude Code
- OpenAI Codex CLI
- Gemini CLI
- Cursor
- GitHub Copilot
- Windsurf
- OpenCode

## Skills 目录结构

### 简单 Skill

```
skill-name/
└── SKILL.md
```

### 完整 Skill

```
skill-name/
├── SKILL.md              # 主指令（必需）
├── README.md             # 说明文档
├── reference.md          # 详细参考
├── examples.md           # 使用示例
├── templates/            # 模板文件
│   ├── component.tsx
│   └── test.ts
├── scripts/              # 辅助脚本
│   ├── validate.sh
│   └── setup.sh
└── resources/            # 其他资源
    └── schema.json
```

## Frontmatter 完整参考

```markdown
---
name: skill-name                    # 必需：Skill 名称
description: What this skill does   # 必需：用途描述

# 触发控制
disable-model-invocation: false     # true = 仅手动触发
user-invocable: true                # false = 隐藏命令

# 工具限制
allowed-tools: Read, Edit, Bash     # 允许的工具

# 执行上下文
context: default                    # fork = 在子代理中运行

# 其他
version: 1.0.0                      # 版本号
author: Your Name                   # 作者
tags: [testing, automation]         # 标签
---
```

## 高级 Skills 示例

### 智能 Commit

```markdown
---
name: smart-commit
description: Create intelligent git commits with conventional format
disable-model-invocation: true
allowed-tools: Bash, Read
---

创建智能 Git Commit：

## 步骤
1. `git status` 查看变更
2. `git diff --staged` 分析暂存内容
3. `git log --oneline -10` 了解提交风格

## Commit 格式
```
<type>(<scope>): <subject>

<body>

<footer>
```

## Type 类型
- feat: 新功能
- fix: Bug 修复
- docs: 文档
- style: 格式
- refactor: 重构
- perf: 性能
- test: 测试
- chore: 构建/工具
- ci: CI/CD

## 规则
- subject 不超过 50 字符
- body 每行不超过 72 字符
- 使用祈使语气（Add 而非 Added）
- 引用相关 Issue（Closes #123）
```

### API 生成器

```markdown
---
name: api-generator
description: Generate REST API endpoints with validation
allowed-tools: Read, Write, Edit, Bash
---

生成 REST API 端点：

## 输入格式
资源名称: $ARGUMENTS

## 生成内容

### 1. 路由文件
```typescript
// routes/{resource}.ts
import { Router } from 'express';
import * as controller from '../controllers/{resource}';
import { validate } from '../middleware/validate';
import { {resource}Schema } from '../schemas/{resource}';

const router = Router();

router.get('/', controller.list);
router.get('/:id', controller.getById);
router.post('/', validate({resource}Schema.create), controller.create);
router.put('/:id', validate({resource}Schema.update), controller.update);
router.delete('/:id', controller.remove);

export default router;
```

### 2. Controller
### 3. Service
### 4. Schema (Zod/Joi)
### 5. Tests

## 执行
1. 分析现有代码结构
2. 生成所有文件
3. 更新路由索引
4. 运行 lint
```

### PR 审查助手

```markdown
---
name: pr-review
description: Comprehensive PR review checklist
allowed-tools: Bash, Read, Grep
---

## PR 审查清单

### 获取变更
```bash
gh pr diff $ARGUMENTS
```

### 审查项目

#### 1. 代码质量
- [ ] 命名清晰
- [ ] 函数单一职责
- [ ] 无重复代码
- [ ] 错误处理完善

#### 2. 安全性
- [ ] 无硬编码密钥
- [ ] 输入验证
- [ ] 无 SQL 注入
- [ ] 无 XSS 风险

#### 3. 性能
- [ ] 无 N+1 查询
- [ ] 适当缓存
- [ ] 无内存泄漏

#### 4. 测试
- [ ] 新代码有测试
- [ ] 测试覆盖边界
- [ ] 测试可读

#### 5. 文档
- [ ] 复杂逻辑有注释
- [ ] API 变更已记录
- [ ] README 已更新

### 输出格式
```markdown
## PR Review: #123

### Summary
[简要总结]

### Issues Found
1. 🔴 Critical: [描述]
2. 🟡 Warning: [描述]
3. 🔵 Suggestion: [描述]

### Approval Status
[ ] Approved / [ ] Changes Requested
```
```

### 数据库迁移

```markdown
---
name: db-migration
description: Generate database migration files
disable-model-invocation: true
allowed-tools: Read, Write, Bash
---

生成数据库迁移：$ARGUMENTS

## 步骤

### 1. 分析需求
解析迁移描述，确定：
- 新增表/字段
- 修改字段
- 删除字段
- 索引变更

### 2. 生成迁移文件
```bash
# Prisma
npx prisma migrate dev --name $MIGRATION_NAME

# Knex
npx knex migrate:make $MIGRATION_NAME

# TypeORM
npx typeorm migration:create src/migrations/$MIGRATION_NAME
```

### 3. 编写迁移代码

#### Up 迁移
[实现正向迁移]

#### Down 迁移
[实现回滚]

### 4. 验证
```bash
# 运行迁移
npm run db:migrate

# 回滚测试
npm run db:rollback
npm run db:migrate
```

### 5. 更新类型
同步更新 TypeScript 类型定义
```

### 依赖更新

```markdown
---
name: deps-update
description: Safely update project dependencies
disable-model-invocation: true
allowed-tools: Bash, Read, Edit
---

安全更新依赖：

## 步骤

### 1. 检查过时依赖
```bash
npm outdated
# 或
pnpm outdated
```

### 2. 安全审计
```bash
npm audit
```

### 3. 更新策略

#### 补丁版本（安全）
```bash
npm update
```

#### 次要版本（测试后）
```bash
npx npm-check-updates -u --target minor
npm install
npm test
```

#### 主要版本（逐个更新）
```bash
npx npm-check-updates -u -f <package>
npm install
npm test
```

### 4. 验证
- 运行测试套件
- 检查类型错误
- 手动测试关键功能

### 5. 记录变更
更新 CHANGELOG.md
```

## 参数处理

### 单个参数

```markdown
搜索: $ARGUMENTS
```

使用: `/search api error`

### 多参数

```markdown
组件: $ARGUMENTS[0]
类型: $ARGUMENTS[1]
```

使用: `/create-component Button functional`

### 环境变量

```markdown
会话: ${CLAUDE_SESSION_ID}
目录: ${PWD}
用户: ${USER}
```

## 引用支持文件

```markdown
详细说明见 [reference.md](reference.md)

模板文件:
- [component.tsx](templates/component.tsx)
- [test.ts](templates/test.ts)
```

## Skills 安装与分享

### 从 GitHub 安装

```bash
# 克隆到 skills 目录
git clone https://github.com/user/skill-name ~/.claude/skills/skill-name

# 或使用 git submodule
cd .claude/skills
git submodule add https://github.com/user/skill-name
```

### 发布 Skill

1. 创建 GitHub 仓库
2. 包含 SKILL.md 和相关文件
3. 添加 README 说明
4. 添加 `claude-skill` topic

### Skills 市场

| 平台 | 地址 |
|------|------|
| Anthropic 官方 | https://github.com/anthropics/skills |
| SkillsMP 市场 | https://skillsmp.com/ |
| Awesome Skills | https://github.com/VoltAgent/awesome-agent-skills |

## 最佳实践

1. **渐进式复杂度** - 从简单开始，逐步添加功能
2. **清晰的描述** - 帮助自动触发
3. **合理的工具限制** - 只给必要权限
4. **包含示例** - 展示预期输入输出
5. **版本控制** - 跟踪变更
6. **测试** - 手动验证后再依赖
7. **文档** - 写清楚如何使用
