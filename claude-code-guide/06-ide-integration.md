# IDE 集成

## VSCode 扩展

### 安装

1. 打开 VSCode
2. 进入扩展市场 (`Ctrl+Shift+X`)
3. 搜索 "Claude Code"
4. 点击安装官方扩展

或使用命令行：

```bash
code --install-extension anthropic.claude-code
```

### 主要功能

- 在 VSCode 内嵌使用 Claude
- 自动差异视图显示代码更改
- Git 集成（创建 commit/PR）
- 文件和选区引用
- 终端输出捕获

### 快捷键

| 快捷键 | 功能 |
|--------|------|
| `Cmd+Esc` (Mac) / `Ctrl+Esc` (Win) | 打开 Claude Code |
| `Cmd+P` / `Ctrl+P` | 模型选择器 |
| `Ctrl+G` | 在外部编辑器打开提示 |
| `Ctrl+S` | 保存当前提示 |
| `Cmd+T` / `Ctrl+T` | 切换扩展思考 |

### 从终端启动

```bash
# 在 VSCode 中启动
claude --ide

# 启动后自动打开 VSCode
code . && claude --ide
```

### 配置

在 VSCode 设置 (`Ctrl+,`) 中搜索 "Claude Code":

- **Claude Code: Default Model** - 默认使用的模型
- **Claude Code: Permission Mode** - 默认权限模式
- **Claude Code: Theme** - 界面主题

### 引用编辑器内容

在对话中直接引用：

```
@当前文件          # 引用打开的文件
@选中的代码        # 引用选区
@终端输出          # 引用终端输出
```

## JetBrains IDE

支持 IntelliJ IDEA、PyCharm、WebStorm、GoLand 等。

### 安装

1. 打开 IDE
2. 进入 Settings → Plugins
3. 搜索 "Claude Code"
4. 点击 Install
5. 重启 IDE

### 快捷键

| 快捷键 | 功能 |
|--------|------|
| `Cmd+Esc` (Mac) / `Ctrl+Esc` (Win/Linux) | 打开 Claude |
| `Cmd+Option+K` (Mac) / `Alt+Ctrl+K` (Win/Linux) | 插入文件引用 |

### 配置

Settings → Tools → Claude Code [Beta]:

- **Claude command**: 指定 Claude 可执行路径
- **Default model**: 默认模型
- **Permission mode**: 权限模式

### WSL 支持

如果在 WSL 中使用，设置自定义命令：

```
wsl -d Ubuntu -- bash -lic "claude"
```

### 常见问题

**ESC 键不工作**

检查 Settings → Terminal，确保没有冲突的快捷键绑定。

## Neovim

### 安装插件

使用 lazy.nvim：

```lua
{
  "anthropics/claude-code.nvim",
  config = function()
    require("claude-code").setup({
      model = "sonnet",
      permission_mode = "acceptEdits"
    })
  end
}
```

使用 packer.nvim：

```lua
use {
  "anthropics/claude-code.nvim",
  config = function()
    require("claude-code").setup()
  end
}
```

### 快捷键

```lua
vim.keymap.set("n", "<leader>cc", ":ClaudeChat<CR>", { desc = "Open Claude Chat" })
vim.keymap.set("v", "<leader>ce", ":ClaudeExplain<CR>", { desc = "Explain selection" })
vim.keymap.set("v", "<leader>cr", ":ClaudeRefactor<CR>", { desc = "Refactor selection" })
```

## 终端集成

### iTerm2 (macOS)

创建快捷键打开 Claude：

1. Preferences → Keys → Hotkeys
2. 创建专用热键窗口
3. 配置命令: `claude`

### Windows Terminal

在 profiles.json 中添加：

```json
{
  "profiles": {
    "list": [
      {
        "name": "Claude Code",
        "commandline": "claude",
        "icon": "path/to/claude-icon.png",
        "startingDirectory": "%USERPROFILE%"
      }
    ]
  }
}
```

### tmux

在 `.tmux.conf` 中添加：

```bash
# 快捷键打开 Claude 窗口
bind-key C new-window -n claude "claude"

# 在新面板中打开
bind-key c split-window -h "claude"
```

## 通用配置

### 环境变量

确保 Claude 在 IDE 终端中可用：

```bash
# ~/.bashrc 或 ~/.zshrc
export PATH="$PATH:/path/to/claude"
```

### 项目级配置

在项目根目录创建 `.claude/settings.json`：

```json
{
  "model": "sonnet",
  "permissions": {
    "defaultMode": "acceptEdits"
  }
}
```

这样无论使用哪个 IDE，配置都会生效。

## 最佳实践

1. **使用原生扩展** - 比终端集成体验更好
2. **配置快捷键** - 快速访问 Claude
3. **利用选区引用** - 选中代码后询问 Claude
4. **保持会话** - 使用 `-c` 继续之前的对话
5. **结合 Git** - 让 Claude 帮助创建 commit 和 PR
