# IndyDevDan: pi-vs-claude-code-main 极速上手手册

这是一个针对 **[Pi Coding Agent](https://github.com/mariozechner/pi-coding-agent)** 的性能加强包，展示了如何通过插件（Extensions）将 Pi 打造为超越 Claude Code 的生产力工具。

---

## 1. 核心铁三角 (工具链)
*   **`pi`**：主体引擎。所有的代码编写、文件修改都是由它完成的。
*   **`Extensions` (扩展)**：Indy 项目的精髓。15+ 个 `.ts` 文件，每个都能给 Pi 增加超能力（如 UI 增强、多Agent协作）。
*   **`just`**：命令运行器。类似于 `Makefile` 或 `npm scripts`，用于封装复杂的叠加命令。

---

## 2. 核心工作流 (启动方式)

### A. 手动加载 (开发/调试)
```bash
pi -e extensions/theme-cycler.ts -e extensions/minimal.ts
```
> **知识点**：`-e` 是可以无限叠加的，允许像拼乐高一样组合功能。

### B. 一键启动 (生产/使用)
```bash
just ext-minimal
```
> **知识点**：`just` 会自动加载 `.env` 密钥，并敲好那一长串叠加的选项。

---

## 3. 三大必装“超能力”插件
1.  **`minimal.ts`** (强烈推荐)：将状态栏简化为 `[###-------] 30%` 进度条，直观监控 Context 消耗。
2.  **`theme-cycler.ts`**：支持快捷键 `Ctrl+X` 实时切肤，让编码环境不再枯燥。
3.  **`damage-control.ts`** (安全卫士)：实时审计 AI 行为，拦截 `rm -rf` 或敏感文件（如 `.env`）的修改。

---

## 4. 关键配置文件意义
*   **`.env`**：密钥仓库。由 `just` 工具自动注入 shell 环境。
*   **`themeMap.ts`**：环境美学字典。定义了不同插件运行时的专属配色方案。
*   **`justfile`**：启动控制台。不仅仅是备忘录，更是可执行的指令集。

---

## 5. 开发者核心概念：`session_start`
*   **定义**：Pi 会话正式开启、UI 准备就绪的瞬间触发。
*   **作用**：用于“开机初始化”，如：同步状态栏、设定初始皮肤、注入系统提示词。

---

## 💡 实验室最佳实践 (Docker)
1.  **保持位置**：库文件（如 `themeMap.ts`）应留在原位。若自动扫描报错，在末尾加个空的 `export default function(){}` 即可。
2.  **双层 Just**：建议在宿主机（Windows）装 Just 用于启动容器，在容器（Docker）里装 Just 用于执行 Indy 的预设指令。
3.  **遵循 CLAUDE.md**：如果让 AI 助理帮你写扩展，先让它读取该文件以保持代码风格统一。

---

*Last Updated: 2026-04-15*
