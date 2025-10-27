# venv + pip-tools 的工作流

## Why pip-tools?

* `pip freeze` 的问题： 它会把“直接依赖”（如 `django`）和“间接依赖”（如 `django` 依赖的 `asgiref`）混在一起。当你 想升级或删除 `django` 时，你根本不知道该动哪些包。
* `pip-tools` 的解决： 可以让用户只关心“直接依赖”，并自动管理所有“间接依赖”。

## 使用 venv 和 pip-tools 进行专业的依赖管理

这个工作流依赖两个核心文件：

* `requirements.in` (输入): 你来维护。只写项目直接需要的高层包 (e.g., django, requests)。
* `requirements.txt` (输出): 机器来生成。包含所有包（直接+间接）的精确版本，用于 CI 和部署。

## 步骤一：项目初始化

1. 创建项目并进入
   ```bash
   mkdir my_django_project
   cd my_django_project
   ```

2. 