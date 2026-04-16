#### 1. 第一步：触发工作流并定义宏观脚手架

你需要先给我下达类似这样的指令：

> “`/feature-driven-dev`，我们现在要开始执行 Obsidian 里的『阶段 1.1：初始化 PWA 项目』。”

- **AI 会怎么做**：我会立刻阅读铁律（强制 Tailwind，不写原生 CSS 等）。然后我会带你创建一份 `_ai_design/features/feature_pwa_init.md` 契约文档。
- **在这个契约里我们会定死**：新建 `mobile_pwa` 目录、选择 `vite-plugin-pwa` 插件、以及基础的网络请求层（Axios 预留动态 `baseURL` 为日后的 Tailscale 铺路）。这确保了我们在写第一行代码前，就已经为阶段 2（内网穿透）留好了气门芯。

#### 2. 第二步：切分“移动端 Pomodoro 页面”作为独立 Feature

初始化完成后，你发起第二次工作流：

> “`/feature-driven-dev`，我们来做『番茄钟的移动端 UI 和核心交互』功能。”

- **AI 会怎么做**：我会再建一个 `feature_mobile_pomodoro.md`。
- **契约阶段（人类把控）**：你会在这里规定“需要哪些后端 API”（比如 `GET /api/pomodoros/`，`POST /api/pomodoros/start`）。由于你说了要复用现在的 Django 后端大脑，所以这份契约会写明：**必须复用现有的后端接口，绝不新造，如果原接口字段太多，必须在后端写一个 LightSerializer 裁剪后返回。**
- **执行阶段（AI 填血肉）**：我会去 `mobile_pwa/src` 里用纯 Tailwind 帮你写出一个全屏的、屏蔽了地址栏的、极具原生质感的竖屏应用。

#### 3. 第三步：穿透与联调 (Stage 2)

当你的手机上已经跑起这个圆角图标的 PWA 后，你可以再开一个总线，专门处理网络层。

> “`/feature-driven-dev`，我们来改造『前端的网络层对接 Tailscale 虚拟 IP』。”