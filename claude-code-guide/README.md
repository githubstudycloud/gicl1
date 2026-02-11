# Claude Code 完全指南

> 最新版本的 Claude Code CLI 工具安装、配置与使用技巧大全

## 目录

### 基础篇
1. [快速开始](01-quickstart.md) - 安装与基础使用
2. [配置详解](02-configuration.md) - settings.json 完整配置
3. [MCP 服务器](03-mcp-servers.md) - 扩展 Claude 能力
4. [自定义命令](04-slash-commands.md) - Skills 斜杠命令
5. [Hooks 钩子](05-hooks.md) - 自动化工作流
6. [IDE 集成](06-ide-integration.md) - VSCode/JetBrains
7. [模型与思考](07-models.md) - 模型选择与扩展思考
8. [快捷键](08-keybindings.md) - 键盘快捷键大全
9. [高级技巧](09-advanced-tips.md) - 最佳实践与技巧
10. [速查表](10-cheatsheet.md) - 常用命令速查

### 进阶篇
11. [子代理详解](11-agents.md) - Agents 创建与使用
12. [Skills 深度指南](12-skills-deep.md) - 高级 Skills 开发
13. [并行与团队](13-parallel-teams.md) - Git Worktree、Team 模式、Swarm
14. [接入其他大模型](14-other-models.md) - OpenAI/Gemini/本地模型
15. [输出风格配置](15-output-style.md) - 自定义输出格式
16. [资源与社区](16-resources.md) - GitHub 资源、Skills 市场、分享平台

## 什么是 Claude Code

Claude Code 是 Anthropic 官方的命令行 AI 编程助手，可以：

- 直接在终端中与 Claude 对话
- 读取、编辑、创建代码文件
- 执行 shell 命令
- 搜索代码库
- 连接外部工具和服务 (MCP)
- 自动化开发工作流
- 创建自定义 Skills 和 Agents
- 并行处理多个任务

## 系统要求

- **操作系统**: macOS, Linux, 或 Windows (需要 WSL 或 Git Bash)
- **运行时**: Node.js 18+ 或 Bun
- **其他**: Git (推荐)

## 快速安装

```bash
# macOS (Homebrew)
brew install claude

# Linux
curl https://download.claude.com/install.sh | bash

# Windows
winget install Anthropic.ClaudeCode

# 验证安装
claude --version
```

## 快速开始

```bash
# 启动交互会话
claude

# 带提示启动
claude "解释这段代码的作用"

# 继续上次对话
claude -c

# 计划模式（只读）
claude --permission-mode plan
```

## 资源链接

| 资源 | 链接 |
|------|------|
| 官方文档 | https://docs.anthropic.com/claude-code |
| Skills 市场 | https://skillsmp.com/ |
| Awesome 集合 | https://github.com/hesreallyhim/awesome-claude-code |
| 300+ Skills | https://github.com/VoltAgent/awesome-agent-skills |
| 实用技巧 | https://github.com/ykdojo/claude-code-tips |

## 许可证

本指南内容基于 Claude Code 官方文档整理，仅供学习参考。
