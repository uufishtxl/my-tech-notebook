# 🐳 Docker 随身实验室：核心命令与原理备忘录

## 1. 基础启动命令拆解

**基础命令：**
```bash
docker run -it --rm -v "%cd%":/workspace -w /workspace ubuntu:22.04 bash
```

**参数字典：**

| 命令/符号 | 全称解析 | 核心作用说明 |
| :--- | :--- | :--- |
| `docker` | - | Docker 命令行客户端工具，整个容器魔法的入口。 |
| `run` | - | 核心子命令，负责“创建”并“启动”一个新的容器。 |
| `-it` | `-i` (interactive)<br>`-t` (tty) | **进入交互模式**。`i` 保持标准输入（STDIN）打开；`t` 分配伪终端（TTY）。让你能像操作本地电脑一样在命令行里交互。 |
| `--rm` | remove | **阅后即焚**。容器退出（exit）后，自动销毁并清理空间，不留任何系统垃圾。 |
| `-v` | volume | **挂载数据卷**。打通物理机与容器的通道。语法为 `物理机路径:容器内路径`。 |
| `"%cd%"` | Current Directory | Windows 变量，代表你当前命令行所在的文件夹路径。 |
| `:` | - | 挂载语法的分隔符。左边是真实世界（Windows），右边是沙盒世界（Ubuntu）。 |
| `/workspace` | - | 容器内的工作目录。我们凭空捏造的一个挂载点。 |
| `-w` | workdir | **指定起始房间**。容器启动后，直接把你传送到指定的目录下（不加这个，默认在 `/` 根目录）。 |
| `ubuntu:22.04` | 镜像名:标签 | 决定你的底稿是什么。这里用的是官方 22.04 版本的纯净 Ubuntu。 |
| `bash` | - | 容器启动后执行的第一个程序。这里启动了 Bash 终端，供我们下达后续指令。 |

---

## 2. 核心原理解读：“任意门”挂载机制

理解 `-v "%cd%":/workspace` 是驾驭 Docker 的关键，请牢记以下物理图景：

- **世界的主体是纯净的 Ubuntu：**  
  沙盒的根目录（`/`）完全属于 Ubuntu。如果你在沙盒里敲 `cd /` 然后 `ls`，你会看到 `bin`, `etc`, `usr`, `root` 等标准的 Linux 系统文件。这与你的 Windows 毫无关系。
- **“任意门”魔法房间：**  
  当你加上挂载命令并在 Windows 的 `C:\pi-sandbox\` 下运行它时，Docker 在 Ubuntu 系统里新建了一个叫 `/workspace` 的空房间，并在这个房间装了一扇“任意门”。
- **空间穿透：**  
  在沙盒的视角里，它的世界（根目录）依然是 Ubuntu。但当你把文件扔进 `/workspace` 这个特殊的魔法房间时，文件会直接穿透隔离墙，掉进你真实的 Windows 硬盘（`C:\pi-sandbox\`）里。

---

## 3. 终极工作台（全副武装模式）

**实战终极命令：**
```bash
docker run -it --rm -v "%cd%":/workspace -v "%userprofile%\.pi":/root/.pi -e MINIMAX_CN_API_KEY="你的真实密钥" my-pi-lab bash
```

**新增威力拆解：**

- **双重挂载（配置持久化）：**  
  `-v "%userprofile%\.pi":/root/.pi`  
  开启第二扇任意门！将你 Windows 用户目录下的 `.pi` 文件夹，死死锚定在沙盒的 `/root/.pi`。这意味着，即便 `--rm` 销毁了系统，你和 AI 的**对话历史、自定义树（Session Tree）**也会永久保存在物理机上。
- **环境变量注入（免登入）：**  
  `-e MINIMAX_CN_API_KEY="..."`  
  在启动的瞬间，把 API 门票挂在沙盒的墙上。进去之后直接敲 `pi` 就能跑，彻底告别每次手动 `export` 的痛苦。
- **专属引擎：**  
  `my-pi-lab`  
  不再使用简陋的 Ubuntu，而是使用你亲手用 `Dockerfile` 熬制的、预装了 Node.js、Python 和 Pi 的专属系统模具。

---

**使用建议：**  
以后无论开发什么新项目，只需在 Windows 终端进入那个项目文件夹，复制粘贴这句“终极命令”，你就能立刻获得一个零污染、带顶级 AI 助手的全栈开发环境。


```powershell
docker build -t my-pi-lab .
```