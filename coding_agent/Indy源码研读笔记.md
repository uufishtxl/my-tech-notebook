# Pi 扩展开发：Theme-Cycler 与 Indy 源码研读笔记

本笔记整合了 `theme-cycler.ts` 与 `minimal.ts` 的源码分析，涵盖了 Pi 扩展开发的通用模式、TUI 布局逻辑及组件生命周期。

---

## 1. UI 界面与状态管理 (UI & Status)

### 1.1 状态栏更新
- **API**: `ctx.ui.setStatus(id: string, content: string)`
- **核心逻辑**：
    - 每个状态栏项通过唯一的 `id` 进行标识。
    - `content` 是渲染在终端底部的文字。
- **最佳实践**：在 `session_start` 事件中初始化状态栏，确保用户进入时界面信息完整。

### 1.2 通知系统
- **API**: `ctx.ui.notify(message: string, type: "info" | "warning" | "error")`
- **特点**：临时提示，会自动消失，常用于反馈命令执行结果。

---

## 2. 自定义组件架构 (Widget Architecture)

`setWidget` 与 `setFooter` 是 Pi 中展示 UI 的核心段手，允许挂载自定义视图。

### 2.1 注册与生存周期
- **注册名**：例如 `"theme-swatch"`，作为组件的唯一 ID，方便后续更新或删除。
- **工厂函数**：渲染逻辑封装在一个函数中 `(_tui, theme) => ({ ... })`，这允许组件访问闭包内的上下文。
- **卸载**：调用 `ctx.ui.setWidget("ID", undefined)` 即可销毁该组件。

### 2.2 渲染函数核心 `render(width)`
- **返回值**：必须返回 `string[]` 数组。数组中的每个元素代表终端界面上的一行。
- **宽度感知**：`width` 参数由系统实时传入，开发者需要根据该宽度决定是否截断文字。

### 2.3 刷新机制 `invalidate`
- 当组件内部状态变化时，调用该方法通知 UI 系统重新触发 `render`。

### 2.4 资源清理 `dispose`
- **作用**：当底部栏被销毁或者被新的 Footer 替换时，系统会自动调用 `dispose` 函数。
- **场景**：比如在 `minimal.ts` 扩展中，如果你需要清理组件的定时器或者注销相关监听器时，就可以用这个方法。
- **Review 补充**：如果不执行 `dispose` 清理，会导致内存泄漏，使得 `pi` 在长时间运行后变得卡顿。

---

## 3. 交互与命令设计 (Interaction & Commands)

### 3.1 快捷键注册 (`pi.registerShortcut`)
- 支持组合键，如 `ctrl+x`。
- 回调函数通过 `ctx` 获取当前上下文。

### 3.2 命令处理与参数 (`pi.registerCommand`)
- **参数解构**：命令后的字符串通过 `args` 传入。
- **技巧**：使用 `args.trim()` 去除首尾空白，判断用户是否输入了具体参数（如 `/theme simple-dark`）。

### 3.3 TUI 式列表选择
- **API**: `await ctx.ui.select(title, items)`
- **特点**：异步阻塞，会在 TUI 上弹出一个交互式列表，用户选择后返回字符串，未选择则返回 `undefined`。
- **防御性编程**：必须判断 `if (!selected) return`，防止用户按 `Esc` 退出后程序崩溃。

---

## 4. TUI 渲染技术细节 (Layout & Rendering)

### 4.1 终端截断与宽度
- **工具**：`truncateToWidth(text, width)`
- **作用**：将字符串截断，确保在等宽字体环境下不超过指定的显示宽度。

### 4.2 字符级排版 (Padding)
- **单位**：格（Character Cells）。
- **逻辑**：`const pad = " ".repeat(Math.max(1, width - visibleWidth(left) - visibleWidth(right)));`
- **核心工具 `visibleWidth`**：用于过滤掉不可见的 ANSI 颜色转义序列，确保宽度计算的精准。

### 4.3 颜色系统
- **API**: `theme.fg(colorName, text)`
- **来源**：这里的 `colorName`（如 `accent`, `muted`）是主题定义的 key。

---

## 5. 生命周期管理 (Lifecycle)

- **`session_start`**：初始化 UI、加载默认配置、设置状态栏。
- **`session_shutdown`**：清理副作用（如 `clearTimeout`），防止内存泄漏。

---
*Created by Antigravity AI on 2026-04-16*