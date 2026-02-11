# 并行工作与团队模式

## Git Worktree 并行开发

### 什么是 Git Worktree

Git Worktree 允许同时检出多个分支到不同目录，每个目录可以运行独立的 Claude Code 会话。

### 基本使用

```bash
# 创建工作树
git worktree add <path> <branch>

# 示例：创建 feature 分支工作树
git worktree add ../feature-auth -b feature/authentication

# 从现有分支创建
git worktree add ../hotfix-login hotfix/login-bug
```

### 并行开发流程

```bash
# 终端 1: 主分支 - Bug 修复
cd /project/main
claude
> Fix the login timeout bug

# 终端 2: Feature 分支 - 新功能
cd /project/feature-auth
claude
> Implement OAuth2 authentication

# 终端 3: Hotfix - 紧急修复
cd /project/hotfix-login
claude
> Fix critical security issue
```

### 工作树管理

```bash
# 列出所有工作树
git worktree list

# 删除工作树
git worktree remove ../feature-auth

# 清理过期的工作树
git worktree prune
```

### 最佳实践

1. **独立目录** - 每个工作树放在项目外
2. **命名清晰** - 目录名反映分支用途
3. **及时清理** - 合并后删除工作树
4. **独立会话** - 每个工作树用独立 Claude 会话

## Team 并行模式

### 启用团队模式

```bash
/teams
```

### 团队架构

```
Lead Agent (协调者)
├── Worker 1 (前端)
├── Worker 2 (后端)
├── Worker 3 (测试)
└── Worker 4 (文档)
```

### 配置团队

在 `.claude/settings.json` 中：

```json
{
  "teams": {
    "enabled": true,
    "maxWorkers": 4,
    "workerModel": "sonnet",
    "leadModel": "opus"
  }
}
```

### 使用团队模式

```bash
# 启用团队
/teams on

# 分配任务
> Implement user dashboard:
> - Worker 1: Create React components
> - Worker 2: Build API endpoints
> - Worker 3: Write integration tests
> - Worker 4: Update documentation

# 查看进度
/teams status

# 关闭团队
/teams off
```

## 容器隔离并行

### Claude-in-Container 模式

让本地 Claude Code 控制容器内的另一个 Claude Code 实例：

```bash
# 启动容器
docker run -it --name claude-worker \
  -v $(pwd):/workspace \
  anthropic/claude-code

# 本地 Claude 通过 tmux 控制
tmux new-session -d -s claude-worker
tmux send-keys -t claude-worker "docker exec -it claude-worker claude" Enter
```

### 优势

- 完全隔离的实验环境
- 长时间运行不阻塞主会话
- 危险操作不影响本地

## 多会话并行

### 方式 1: 多终端

```bash
# 终端 1
claude --session-id "frontend-work"

# 终端 2
claude --session-id "backend-work"

# 终端 3
claude --session-id "testing"
```

### 方式 2: tmux 多窗口

```bash
# 创建会话
tmux new-session -s dev

# 分割窗口
Ctrl+b %  # 垂直分割
Ctrl+b "  # 水平分割

# 每个窗格运行不同任务
```

### 方式 3: 后台代理

```bash
# 启动后台代理
claude -p "Explore the codebase and document the architecture" &

# 继续前台工作
claude
```

## 并行代理任务

### 同时启动多个代理

```bash
# 并行运行
Launch these agents in parallel:
1. security-auditor to review auth/
2. test-generator to create tests
3. doc-writer to update docs

# 或使用 Task 工具
Use Task tool to launch multiple agents simultaneously
```

### 监控并行任务

```bash
# 查看运行中的任务
/tasks

# 获取任务输出
/tasks output <task-id>

# 等待任务完成
/tasks wait <task-id>
```

## 分布式工作流

### GitHub Actions 集成

```yaml
# .github/workflows/claude-review.yml
name: Claude Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code

      - name: Run Review
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          claude -p "Review this PR for issues" < <(git diff origin/main...HEAD)
```

### CI/CD 并行任务

```yaml
jobs:
  claude-tasks:
    strategy:
      matrix:
        task: [security-audit, code-review, doc-check]
    steps:
      - run: claude -p "/${{ matrix.task }}" --max-turns 10
```

## Swarm 模式（实验性）

### 什么是 Swarm

Swarm 是多个代理协作完成复杂任务的模式。

### 启用 Swarm

```bash
# 在计划模式中
/plan
> Design a complete authentication system

# 退出计划时选择 Swarm
/exitplan --launchSwarm --teammateCount 3
```

### Swarm 工作流

```
1. Lead Agent 分解任务
2. 分配给 Worker Agents
3. Workers 并行执行
4. Lead Agent 整合结果
5. 验证和修正
```

## 最佳实践

### 1. 任务分解

```bash
# 好的分解
- Frontend: Components only
- Backend: API only
- Database: Migrations only

# 避免的分解
- All of frontend (太大)
- Random files (无逻辑)
```

### 2. 避免冲突

```bash
# 分配不重叠的文件/目录
Worker 1: src/components/
Worker 2: src/api/
Worker 3: src/utils/
```

### 3. 同步点

```bash
# 在关键节点同步
> Wait for all workers to complete the models
> Then proceed with integration
```

### 4. 上下文共享

```bash
# 共享的 CLAUDE.md
# 每个工作树都能访问相同的项目规范
```

### 5. 资源管理

```bash
# 监控 API 使用
/cost

# 限制并行数量
# 太多并行会导致 rate limiting
```

## 实用场景

### 场景 1: 功能开发

```bash
# Worktree 1: UI
git worktree add ../ui-work -b feature/new-ui
cd ../ui-work && claude
> Create the new dashboard UI

# Worktree 2: API
git worktree add ../api-work -b feature/new-api
cd ../api-work && claude
> Implement the dashboard API
```

### 场景 2: 代码审查

```bash
# 并行审查不同方面
Use agents in parallel:
1. security-auditor for security
2. performance-optimizer for performance
3. code-reviewer for code quality
```

### 场景 3: 大规模重构

```bash
# 分而治之
/teams on
> Refactor the monolith into microservices:
> - Worker 1: Extract user service
> - Worker 2: Extract order service
> - Worker 3: Extract payment service
> - Worker 4: Update API gateway
```
