# AI CLI 工具接口格式分析

本文档分析主流 AI CLI 工具和会话接口格式，研究它们之间的相互转换方案。

## 目录

1. [概述](#概述)
2. [工具对比表](#工具对比表)
3. [各工具详细分析](#各工具详细分析)
4. [通用 API 格式](#通用-api-格式)
5. [接口转换方案](#接口转换方案)

---

## 概述

| 工具 | 开发者 | 协议 | 主要特点 |
|------|--------|------|----------|
| Claude Code | Anthropic | Anthropic Messages API | OpenAI 兼容网关、MCP 支持 |
| Codex CLI | OpenAI | JSON-RPC 2.0 (JSONL/stdio) | App Server、MCP 支持 |
| Gemini CLI | Google | Gemini API (REST/SSE) | ReAct 循环、MCP 支持 |
| OpenCode | 社区 | OpenAI 兼容 + ACP | 多 Provider、HTTP Server |

---

## 工具对比表

### 输入/输出格式对比

| 特性 | Claude Code | Codex CLI | Gemini CLI | OpenCode |
|------|-------------|-----------|------------|----------|
| 输出格式 | JSON/Text | JSONL/JSON | JSON/Stream-JSON | JSON/Text |
| 配置格式 | TOML/JSON | TOML | YAML/JSON | JSON/JSONC |
| 流式支持 | ✓ | ✓ | ✓ (SSE) | ✓ |
| MCP 支持 | ✓ | ✓ | ✓ | ✓ |
| 工具调用 | ✓ | ✓ | ✓ | ✓ |

---

## 各工具详细分析

详见各子文档：
- [claude-code.md](./claude-code.md) - Claude Code 接口分析
- [codex-cli.md](./codex-cli.md) - Codex CLI 接口分析
- [gemini-cli.md](./gemini-cli.md) - Gemini CLI 接口分析
- [opencode.md](./opencode.md) - OpenCode 接口分析
- [api-formats.md](./api-formats.md) - 通用 API 格式对比
- [conversion.md](./conversion.md) - 接口转换方案

---

## 参考资料

- [Claude Code CLI Reference](https://code.claude.com/docs/en/cli-reference)
- [OpenAI Codex CLI](https://developers.openai.com/codex/cli)
- [Google Gemini CLI](https://github.com/google-gemini/gemini-cli)
- [OpenCode](https://opencode.ai/docs/)
- [OpenAI Chat Completions API](https://platform.openai.com/docs/api-reference/chat)
- [Anthropic Claude Messages API](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)
