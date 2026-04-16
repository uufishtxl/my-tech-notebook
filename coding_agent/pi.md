# 终端图形界面
- CLI: Command Line Interface: 简单的问答模式
- TUI: Text User Interface, 在终端里运行的图形界面
	- Layout: Top: Status Bar / Middle: Editor / Bottom: Message box
	- Palette (ANSI)
	- Interaction (Mouse / Shortcut Behaviors)
在 IndyDevDan 的主题插件中，TUI 的体现：
- `ctx.ui.setStatus` - 在终端底部或顶部显示当前主题
- `ctx.ui.setWidget` - 弹出一个临时的彩色色块组件
- `theme.fg("accent", block)` - 给文本上色
# `ctx` - Extension Context

`ctx` 是插件的“万能通行证”或者“当前环境的快照”。当在插件里注册一个快捷键或命令时，回调函数都会带上这个 `ctx`，它主要包含两个核心功能：
- 感知环境 (`ctx.hasUI`)：它告诉你现在是在纯文本终端运行，还是在一个支持图形交互的 TUI 模式下
- 操控 UI (`ctx.ui`)：
	- **通知**：`ctx.notify("msg text", "info"`)，就像网页的 Toast 弹窗
	- **选择**：`ctx.ui.select("title", list`)，弹出一个列表让你选择主题，就像 HTML 的 `<select>`
	- **主题控制**：`ctx.ui.setTheme(name)` —— 核心功能，告诉主程序“换件衣服”。
	- **状态维护**：`ctx.ui.setStatus` —— 在状态栏持久显示一些信息。Mario 在 `@mariozechner/pi-coding-agent` 这个底层库里，提供了一个原生的能力：**“允许扩展在屏幕最底部的状态栏上贴小纸条”**。这个能力就是 `ctx.ui.setStatus` 方法。
		![[pi_status_theme.png|278]]
`ctx` 是 Agent 的**运行环境中心对象**，它像一个“中控枢纽”，集成了 **ui**（交互）、**fs**（文件系统）、**llm**（模型能力）和 **tools**（工具集）等核心模块。


![[ctx_ui_select.png]]

它让插件和 Agent 在执行任务时，能够通过这一个入口快速调用系统的各种底层能力。

- **ctx.ui**：负责所有用户交互界面，包括打印消息、显示进度条、弹出确认框以及管理终端主题（Themes）。
    
- **ctx.fs**：提供受限的文件系统访问，允许 Agent 在当前项目工作区内进行文件的读取、写入、搜索和删除操作。
    
- **ctx.llm**：作为大语言模型的统一接口，负责构造 Prompt、发送请求并解析返回结果，支持流式输出和 Token 计数。
    
- **ctx.tools**：工具管理中心，负责注册、查找和执行 Agent 可以调用的具体功能函数（如运行终端命令、解析代码等）。
    
- **ctx.project**：维护当前项目的元数据，包括项目路径、忽略文件规则以及通过扫描得到的文件树结构。
    
- **ctx.config**：存储和获取全局配置或当前会话的特定设置，例如模型参数、API 密钥或插件自定义选项。
---

### session_start

- 这是 Agent 生命周期中的**启动事件**，在新的会话建立时被触发。
    
- 它主要负责**初始化工作**，比如加载历史记录、配置环境变量以及让 Agent 准备好接收第一个任务。
    
- 你可以把它理解为 Agent 的“开机自检”和“开场白”触发器。