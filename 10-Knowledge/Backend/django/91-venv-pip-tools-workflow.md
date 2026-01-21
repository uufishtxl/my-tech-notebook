# venv + pip-tools 的工作流

## Why pip-tools?

* `pip freeze` 的问题： 它会把“直接依赖”（如 `django`）和“间接依赖”（如 `django` 依赖的 `asgiref`）混在一起。当你 想升级或删除 `django` 时，你根本不知道该动哪些包。
* `pip-tools` 的解决： 可以让用户只关心“直接依赖”，并自动管理所有“间接依赖”。

## 使用 venv 和 pip-tools 进行专业的依赖管理

这个工作流依赖两个核心文件：

* `requirements.in` (输入): 你来维护。只写项目直接需要的高层包 (e.g., django, requests)。
* `requirements.txt` (输出): 机器来生成。包含所有包（直接+间接）的精确版本，用于 CI 和部署。

## 步骤一：项目初始化

1. 创建项目并进入：
   ```bash
   mkdir my_django_project
   cd my_django_project
   ```

2. 创建虚拟环境：
   ```bash
   python -m venv venv
   ```

3. 激活虚拟环境：
    ```bash
    # macOS / Linux
    source venv/bin/activate
    # Windows
    .\venv\Scripts\activate
    ```

   > 🎯: 推荐在 PowerShell 中运行此命令。如果提示禁用脚本，请执行 `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser` 命令，以启用脚本运行功能。

4. 安装 pip-tools 在这个干净的 *venv* 中，第一个（也几乎是唯一一个）手动安装的包就是`pip-tools`。
    ```bash
    pip install pip-tools
    ```

5. 手动创建 `requirements.in`文件。
    ```bash
    touch requirements.in
    ```

6. 打开 `requirements.in`，写入：
    ```Ini, TOML
    # requirements.in
    django
    python-decouple
    # ...
    ```

7. 编译并安装。
    ```Ini, TOM
    # 1. 编译：读取 .in，生成 .txt
    pip-compile requirements.in

    # 2. 同步：读取 .txt，安装所有包
    pip-sync
    ```
    在 `pip-sync` 这一步，Django (以及 `python-decouple` 等) 才被真正安装到你的 venv 中。

8. 运行 `django-admin`创建项目并继续。
   ```bash
   django-admin startproject my_sample_project .
   ```

   **为什么要用 . 结尾？** 这告诉 `django-admin`：“不要再创建一个新的 `my_sample_project` 文件夹了，就在当前目录 (`.`) 把项目文件（`manage.py` 和 `my_sample_project/` 文件夹）创建出来。”

   最终项目结构：

   ```
   my_awesome_project/
    ├── venv/                 <-- 你的虚拟环境
    ├── config/               <-- Django 项目配置文件夹 (settings.py 在这里)
    ├── manage.py             <-- Django 核心管理脚本
    ├── requirements.in       <-- 【你创建的】依赖蓝图 (和 manage.py 是邻居)
    └── requirements.txt      <-- 【机器生成的】锁定依赖 (和 manage.py 是邻居)
   ```
## 添加/删除包的情景

1. 打开`requirements.in`文件，在最后一行加入包的名称。

2. 运行 `pip-compile requirements.in`,`pip-tools`会找到新的包和它的子依赖，并将它们全部锁定版本的状态写入 `requirements.txt`。

3. 同步环境（Sync）：运行 `pip-sync`。`pip-sync`会对比 `requirements.txt` 和当前的 `venv`，发现 `requests`及其子依赖的缺失，自动安装它们。

* 删除场景也类似，只需要将移除的包的名称从`requirements.in`中移除，然后执行后面的两个步骤即可。