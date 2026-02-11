# 键盘快捷键

## 全局快捷键

| 快捷键 | 功能 |
|--------|------|
| `Ctrl+C` | 中断当前操作 |
| `Ctrl+D` | 退出 Claude |
| `Ctrl+T` | 切换任务列表 |
| `Ctrl+O` | 切换详细日志 (verbose) |
| `Ctrl+R` | 搜索历史 |
| `Ctrl+L` | 清屏 |

## 聊天输入快捷键

| 快捷键 | 功能 |
|--------|------|
| `Enter` | 提交消息 |
| `Escape` | 取消输入/返回 |
| `Ctrl+G` | 在外部编辑器打开 |
| `Ctrl+S` | 保存提示 |
| `Ctrl+V` | 粘贴图像 |
| `Shift+Tab` | 循环权限模式 |
| `Tab` | 自动补全 |

## 模型和思考

| 快捷键 | 功能 |
|--------|------|
| `Cmd+P` / `Meta+P` | 模型选择器 |
| `Cmd+T` / `Meta+T` | 切换扩展思考 |
| `Option+T` / `Alt+T` | 切换扩展思考 |

## 导航快捷键

| 快捷键 | 功能 |
|--------|------|
| `↑` / `↓` | 浏览历史 |
| `Page Up` / `Page Down` | 滚动输出 |
| `Home` / `End` | 跳到开头/结尾 |

## 自定义快捷键

### 配置文件

编辑 `~/.claude/keybindings.json`:

```json
{
  "$schema": "https://www.schemastore.org/claude-code-keybindings.json",
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+e": "chat:externalEditor",
        "ctrl+u": null,
        "shift+enter": "chat:submit"
      }
    },
    {
      "context": "Global",
      "bindings": {
        "ctrl+shift+r": "history:search"
      }
    }
  ]
}
```

### 可用上下文

| 上下文 | 说明 |
|--------|------|
| `Global` | 全局生效 |
| `Chat` | 聊天输入时 |
| `Autocomplete` | 自动补全菜单 |
| `Confirmation` | 确认对话框 |
| `Tabs` | 标签导航 |
| `Transcript` | 对话记录查看 |
| `HistorySearch` | 历史搜索 |
| `Task` | 任务视图 |
| `Help` | 帮助页面 |
| `Settings` | 设置页面 |
| `Plugin` | 插件页面 |
| `Select` | 选择菜单 |
| `DiffDialog` | 差异对话框 |
| `ModelPicker` | 模型选择器 |

### 可用动作

**Chat 上下文**:
- `chat:submit` - 提交消息
- `chat:externalEditor` - 在外部编辑器打开
- `chat:save` - 保存提示
- `chat:paste` - 粘贴
- `chat:cancel` - 取消

**Global 上下文**:
- `history:search` - 搜索历史
- `task:toggle` - 切换任务列表
- `verbose:toggle` - 切换详细日志
- `exit` - 退出

### 解除绑定

设置为 `null` 解除默认绑定：

```json
{
  "context": "Chat",
  "bindings": {
    "ctrl+u": null
  }
}
```

## 快捷键示例配置

### Vim 风格

```json
{
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+j": "chat:submit",
        "ctrl+c": "chat:cancel",
        "ctrl+o": "chat:externalEditor"
      }
    },
    {
      "context": "Global",
      "bindings": {
        "ctrl+n": "history:next",
        "ctrl+p": "history:previous"
      }
    }
  ]
}
```

### Emacs 风格

```json
{
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+x ctrl+s": "chat:save",
        "ctrl+x ctrl+c": "exit",
        "meta+x": "command:palette"
      }
    }
  ]
}
```

## IDE 快捷键

### VSCode

| 快捷键 | 功能 |
|--------|------|
| `Cmd+Esc` (Mac) | 打开 Claude Code |
| `Ctrl+Esc` (Win) | 打开 Claude Code |
| `Cmd+P` | 模型选择器 |
| `Ctrl+G` | 外部编辑器 |

### JetBrains

| 快捷键 | 功能 |
|--------|------|
| `Cmd+Esc` (Mac) | 打开 Claude |
| `Ctrl+Esc` (Win/Linux) | 打开 Claude |
| `Cmd+Option+K` (Mac) | 插入文件引用 |
| `Alt+Ctrl+K` (Win/Linux) | 插入文件引用 |

## 快捷键管理命令

```bash
# 打开快捷键配置
/keybindings

# 查看当前绑定
/help keybindings
```

## 常用组合

### 高效编码流程

```
1. Ctrl+G     → 在编辑器写长提示
2. Enter      → 提交
3. Shift+Tab  → 切换到 acceptEdits 模式
4. Ctrl+T     → 打开任务列表跟踪进度
```

### 快速调试

```
1. Cmd+P      → 切换到 opus 模型
2. Option+T   → 启用扩展思考
3. 粘贴错误日志
4. Enter      → 提交
```

### 代码审查

```
1. @file.ts   → 引用文件
2. Ctrl+G     → 编辑详细审查要求
3. Enter      → 提交
4. Ctrl+S     → 保存审查结果
```
