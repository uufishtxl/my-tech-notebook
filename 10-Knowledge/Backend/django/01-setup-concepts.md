# 页面一：安装、概念与项目启动

## 1. 核心 Web 框架概念

* **MVC (Model-View-Controller):** 经典 Web 框架模式。
* **MVT (Model-View-Template):** Django 的自称。
    * **Model (模型):** 数据和数据库。
    * **View (视图):** 处理用户交互（接收请求，发送响应）。
    * **Template (模板):** HTML 标记语言，负责展示。
* [点击查看 MVC, MVP, MVVM 对比](../../software-design-patterns/mvc-mvp-mvvm-patterns.md)

## 2. 环境安装

1.  **安装 Django：** `pip install django`
    * *笔记：* 建议在虚拟环境中安装。

2.  **虚拟环境：**
    * **入门课：** `pipenv shell`
    * **更好的选择 (推荐):** 使用 Python 内置的 `venv`
        ```bash
        # 1. 创建 venv
        python -m venv venv
        # 2. 激活 (macOS/Linux)
        source venv/bin/activate
        # 2. 激活 (Windows)
        .\venv\Scripts\activate
        ```

## 3. 启动项目流程

1.  **创建项目 (Project)：**
    * `django-admin startproject <project_name> .`
    * *注意：末尾的 `.` 很重要，它避免了多一层目录嵌套。*

2.  **创建应用 (App)：**
    * `python manage.py startapp <app_name>`

3.  **注册 App：**
    * 打开 `<project_name>/settings.py`。
    * 将你的 `<app_name>` 添加到 `INSTALLED_APPS` 列表中。

4.  **运行项目：**
    * `python manage.py runserver`
    * 访问 `http://127.0.0.1:8000/` 查看。