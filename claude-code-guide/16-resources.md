# GitHub 资源与社区

## Awesome 资源集合

### 综合资源

| 仓库 | 说明 | Stars |
|------|------|-------|
| [hesreallyhim/awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) | Skills, Hooks, 插件大全 | ⭐⭐⭐ |
| [jqueryscript/awesome-claude-code](https://github.com/jqueryscript/awesome-claude-code) | 工具、IDE、框架集合 | ⭐⭐⭐ |
| [affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code) | Anthropic 黑客松冠军的完整配置 | ⭐⭐⭐ |
| [ykdojo/claude-code-tips](https://github.com/ykdojo/claude-code-tips) | 45+ 实用技巧，含 dx 插件 | ⭐⭐⭐ |

### Skills 专项

| 仓库 | 说明 |
|------|------|
| [anthropics/skills](https://github.com/anthropics/skills) | Anthropic 官方 Skills 仓库 |
| [VoltAgent/awesome-agent-skills](https://github.com/VoltAgent/awesome-agent-skills) | 300+ Skills，兼容多平台 |
| [travisvn/awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills) | 精选 Skills 列表 |
| [ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills) | 实用 Skills 集合 |

## Skills 市场与分享平台

| 平台 | 地址 | 说明 |
|------|------|------|
| **SkillsMP** | https://skillsmp.com/ | Skills 市场，搜索和分类 |
| **Awesome Claude** | https://awesomeclaude.ai/ | Claude 资源目录 |
| **Anthropic Skills** | https://github.com/anthropics/skills | 官方 Skills |

## 精选技巧来源

### Advent of Claude (Ado's Tips)

Anthropic DevRel Ado (@adocomplete) 在 2025 年 12 月发布的每日技巧系列：

- 上下文管理最佳实践
- MCP 优化技巧
- Skills 高效使用
- 容器隔离并行

### ykdojo/claude-code-tips 重点内容

1. **dx 插件** - 开发体验增强
   - `/dx:clone` - 智能克隆
   - `/dx:half-clone` - 精简克隆
   - `/dx:handoff` - 会话交接
   - `/dx:gha` - GitHub Actions 辅助

2. **系统提示减半** - 优化 token 使用

3. **Gemini CLI 作为 Minion** - 混合使用

4. **容器自运行** - Claude 在容器中自主运行

## 官方资源

| 资源 | 地址 |
|------|------|
| 官方文档 | https://docs.anthropic.com/claude-code |
| GitHub 仓库 | https://github.com/anthropics/claude-code |
| 问题反馈 | https://github.com/anthropics/claude-code/issues |
| Discord 社区 | https://discord.gg/anthropic |

## MCP 服务器资源

| 仓库 | 说明 |
|------|------|
| [anthropics/mcp-servers](https://github.com/anthropics/mcp-servers) | 官方 MCP 服务器 |
| [modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers) | MCP 协议服务器集合 |

### 热门 MCP 服务器

- **mcp-server-github** - GitHub 集成
- **mcp-server-postgres** - PostgreSQL 数据库
- **mcp-server-slack** - Slack 集成
- **mcp-server-filesystem** - 文件系统访问
- **mcp-server-chrome** - 浏览器控制
- **mcp-server-sentry** - 错误监控
- **context7** - 文档查询

## 插件与扩展

### IDE 插件

| IDE | 插件 |
|-----|------|
| VSCode | [Claude Code Extension](https://marketplace.visualstudio.com/items?itemName=anthropic.claude-code) |
| JetBrains | Claude Code Plugin |
| Neovim | claude-code.nvim |

### CLI 增强

| 工具 | 说明 |
|------|------|
| [dx plugin](https://github.com/ykdojo/claude-code-tips) | 开发体验增强 |
| [claude-code-proxy](https://github.com/1rgs/claude-code-proxy) | 多模型代理 |

## 社区精选配置

### everything-claude-code 配置集

来自 Anthropic 黑客松冠军，包含：

- **15+ Agents** - 预配置子代理
- **30+ Skills** - 实用技能
- **30+ Commands** - 自定义命令
- **Hooks** - 自动化钩子
- **MCP 配置** - 服务器配置

GitHub: [affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code)

### 安装方式

```bash
# 克隆配置
git clone https://github.com/affaan-m/everything-claude-code ~/.claude-config

# 链接配置
ln -s ~/.claude-config/.claude ~/.claude
```

## 跨平台 Skills 兼容性

2025 年底，Skills 格式成为开放标准，以下工具都支持：

| 工具 | 支持情况 |
|------|----------|
| Claude Code | ✅ 完整支持 |
| OpenAI Codex CLI | ✅ 采用相同格式 |
| Gemini CLI | ✅ 兼容 |
| Cursor | ✅ 兼容 |
| GitHub Copilot | ✅ 兼容 |
| Windsurf | ✅ 兼容 |
| OpenCode | ✅ 兼容 |

## 学习资源

### 文章与教程

| 标题 | 来源 |
|------|------|
| [Claude Agent Skills 深入分析](https://leehanchung.github.io/blogs/2025/10/26/claude-skills-deep-dive/) | Lee Han Chung |
| [Equipping Agents with Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) | Anthropic Engineering |
| [Skills vs Commands vs Subagents](https://www.youngleaders.tech/p/claude-skills-commands-subagents-plugins) | Young Leaders Tech |

### 视频教程

- YouTube: 搜索 "Claude Code tutorial"
- Anthropic 官方频道

## 实用技巧汇总

### 上下文优化

```
不要同时启用所有 MCP！
200k 上下文可能缩减到 70k
```

### Skills 效率

```
Skills 使用渐进式加载：
- 元数据扫描：~100 tokens/skill
- 激活加载：<5k tokens
```

### 容器隔离

```bash
# 让本地 Claude 控制容器中的 Claude
# 实现完全自主的 worker
docker run -it anthropic/claude-code
```

## 贡献与分享

### 分享你的 Skill

1. 创建 GitHub 仓库
2. 包含 `SKILL.md` 和相关文件
3. 添加 `README.md` 说明
4. 添加 topic: `claude-skill`, `agent-skill`
5. 提交到 awesome 列表

### 分享你的 Agent

1. 创建配置文件 `agent-name.md`
2. 测试功能
3. 编写文档
4. 分享到社区

## 保持更新

### 关注渠道

- **GitHub**: Watch anthropics/claude-code
- **Twitter/X**: @AnthropicAI
- **Discord**: Anthropic 社区
- **Reddit**: r/ClaudeAI

### 更新命令

```bash
# 检查更新
claude --version

# 更新
claude update

# 或重新安装
brew upgrade claude  # macOS
```

## 快速链接汇总

```
官方文档:     https://docs.anthropic.com/claude-code
官方 Skills:  https://github.com/anthropics/skills
Skills 市场:  https://skillsmp.com/
Awesome 集合: https://github.com/hesreallyhim/awesome-claude-code
300+ Skills:  https://github.com/VoltAgent/awesome-agent-skills
实用技巧:     https://github.com/ykdojo/claude-code-tips
完整配置:     https://github.com/affaan-m/everything-claude-code
MCP 服务器:   https://github.com/anthropics/mcp-servers
```
