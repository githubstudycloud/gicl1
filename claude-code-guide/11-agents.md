# 子代理 (Agents) 详解

## 什么是子代理

子代理是独立运行的 Claude 实例，用于处理特定任务。它们：

- 有独立的上下文（不污染主会话）
- 可以使用不同的模型
- 可以有不同的权限配置
- 适合并行处理多个任务

## 内置代理类型

Claude Code 提供以下内置代理：

| 代理类型 | 用途 |
|----------|------|
| `Explore` | 快速探索代码库 |
| `Plan` | 设计实现计划 |
| `general-purpose` | 通用任务 |
| `bug-analyzer` | 深度调试分析 |
| `code-reviewer` | 代码审查 |
| `dev-planner` | 开发计划分解 |
| `story-generator` | 生成用户故事 |
| `ui-sketcher` | UI 蓝图设计 |

## 使用内置代理

```bash
# 自然语言调用
Use a subagent to explore how authentication works

# 指定代理类型
Use the code-reviewer subagent to review src/auth/

# 探索代码库
Use the Explore agent to find all API endpoints
```

## 创建自定义代理

### 文件位置

```
~/.claude/agents/           # 用户全局
.claude/agents/             # 项目级
```

### 代理配置格式

`.claude/agents/my-agent.md`:

```markdown
---
name: my-agent
description: 代理用途描述（用于自动选择）
tools: Read, Grep, Glob, Bash, Edit
model: sonnet
permissionMode: default
memory: user
---

代理的系统提示...
```

### Frontmatter 字段

| 字段 | 说明 | 示例 |
|------|------|------|
| `name` | 代理名称 | `security-auditor` |
| `description` | 用途描述 | 用于自动选择 |
| `tools` | 可用工具列表 | `Read, Grep, Edit` |
| `model` | 使用的模型 | `opus`, `sonnet`, `haiku` |
| `permissionMode` | 权限模式 | `default`, `plan`, `acceptEdits` |
| `memory` | 记忆范围 | `user`, `project`, `none` |

## 实用代理示例

### 安全审计代理

`.claude/agents/security-auditor.md`:

```markdown
---
name: security-auditor
description: Expert security auditor for vulnerability detection
tools: Read, Grep, Glob
model: opus
permissionMode: plan
---

你是资深安全审计专家。检查代码中的：

## 漏洞类型
- SQL 注入
- XSS 跨站脚本
- CSRF 跨站请求伪造
- 认证绕过
- 敏感数据泄露
- 不安全的依赖

## 输出格式
按严重程度（Critical/High/Medium/Low）输出报告：
- 文件位置和行号
- 漏洞描述
- 修复建议
- 参考资料
```

### 测试生成代理

`.claude/agents/test-generator.md`:

```markdown
---
name: test-generator
description: Generate comprehensive tests for code
tools: Read, Grep, Glob, Edit, Write
model: sonnet
permissionMode: acceptEdits
---

你是测试工程师。为代码生成全面的测试：

## 测试策略
1. 单元测试：每个函数/方法
2. 边界条件：空值、极值、错误输入
3. 集成测试：模块间交互
4. 模拟：外部依赖

## 测试框架
- JavaScript/TypeScript: Jest/Vitest
- Python: pytest
- Go: testing package

## 输出
- 测试文件放在 __tests__ 或 *_test 目录
- 覆盖率目标 80%+
```

### 文档生成代理

`.claude/agents/doc-writer.md`:

```markdown
---
name: doc-writer
description: Generate documentation for code
tools: Read, Grep, Glob, Write
model: sonnet
permissionMode: acceptEdits
---

你是技术文档专家。生成清晰的文档：

## 文档类型
- README.md: 项目概述
- API 文档: 端点说明
- 代码注释: JSDoc/docstring
- 架构文档: 系统设计

## 风格指南
- 简洁明了
- 包含示例代码
- 保持更新
```

### 重构代理

`.claude/agents/refactorer.md`:

```markdown
---
name: refactorer
description: Refactor code for better quality
tools: Read, Grep, Glob, Edit
model: opus
permissionMode: default
---

你是重构专家。改进代码质量：

## 重构目标
- 提取重复代码
- 简化复杂逻辑
- 改善命名
- 优化性能
- 增强可读性

## 原则
- 保持行为不变
- 小步重构
- 每步可测试
- 不破坏 API
```

### 性能优化代理

`.claude/agents/performance-optimizer.md`:

```markdown
---
name: performance-optimizer
description: Analyze and optimize code performance
tools: Read, Grep, Glob, Bash, Edit
model: opus
permissionMode: default
---

你是性能优化专家。

## 分析项目
- 算法复杂度
- 内存使用
- I/O 瓶颈
- 数据库查询
- 网络请求

## 优化策略
- 缓存热点数据
- 延迟加载
- 并行处理
- 减少不必要计算
- 优化数据结构

## 输出
- 性能问题清单
- 优化建议
- 预期改进
```

## 代理使用技巧

### 自动委托

Claude 会根据 description 自动选择合适的代理：

```bash
# Claude 自动选择 security-auditor
Review this code for security issues

# Claude 自动选择 test-generator
Generate tests for the auth module
```

### 显式调用

```bash
# 指定代理名称
Use the refactorer subagent to clean up src/utils/

# 带参数
Use the doc-writer subagent to document the API endpoints
```

### 并行代理

```bash
# 同时运行多个代理
Launch these agents in parallel:
1. security-auditor to review auth/
2. test-generator to create tests for api/
3. doc-writer to document models/
```

## 代理与 Skills 的区别

| 特性 | Agent | Skill |
|------|-------|-------|
| 上下文 | 独立（隔离） | 共享主会话 |
| 模型 | 可指定不同模型 | 使用当前模型 |
| 适用场景 | 复杂独立任务 | 简单可复用流程 |
| Token 消耗 | 独立计算 | 共享上下文 |
| 触发方式 | 自动/手动 | 斜杠命令/自动 |

## 管理代理

```bash
# 查看可用代理
/agents

# 创建新代理（交互式）
/agents new

# 编辑代理
/agents edit agent-name
```

## 最佳实践

1. **单一职责** - 每个代理专注一类任务
2. **选择合适模型** - 复杂任务用 opus，简单用 haiku
3. **限制工具** - 只给必要的权限
4. **写好描述** - 帮助自动选择
5. **测试代理** - 确保行为符合预期
6. **版本控制** - 将代理配置提交到 git
