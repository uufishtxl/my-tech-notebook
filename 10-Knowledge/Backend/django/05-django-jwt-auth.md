# 笔记：搭建 Django DRF 全栈认证 (Auth) 地基

这是在 Django 后端搭建一个现代的、基于 Token (JWT) 的认证 API 的标准流程。我们使用了“三剑客”：`DRF` + `dj-rest-auth` + `simplejwt`。

---

## 1. 核心组件 (The "Lego" Bricks)

我们到底装了什么？

1.  **`djangorestframework` (DRF):**
    * **这是什么？** Django 官方的 API 框架。
    * **它的工作？** 把 Django 从一个“网站服务器”变成一个“API 服务器”。它让 `models` 和 `views` 可以轻松地返回 `JSON` 数据。

2.  **`django-allauth`:**
    * **这是什么？** 一个**极其**强大的“本地认证”包。
    * **它的工作？** 处理所有“账户管理”的*内部逻辑*，比如“用 Email 登录”（而不是用户名）、“注册时验证两次密码”、“重置密码”等。**它是发动机。**

3.  **`dj-rest-auth`:**
    * **这是什么？** `allauth` 的“API 接口层”。
    * **它的工作？** 把 `allauth` 的*内部逻辑*（发动机），**暴露**成我们可以从 Vue 调用的 `/api/auth/login/`, `/api/auth/register/` 接口。

4.  **`djangorestframework-simplejwt`:**
    * **这是什么？** “JWT 令牌”生成器。
    * **它的工作？** 取代了 Django 老旧的 `Session` 认证。当 `dj-rest-auth` 登录成功后，它会调用 `simplejwt` 来生成一个加密的 `Token`（令牌）并返回给前端。

**关系图：**
`Vue (前端)` → `dj-rest-auth (API接口)` → `django-allauth (认证逻辑)` → `simplejwt (令牌生成)`

---

## 2. 搭建步骤

### 步骤 1: 安装依赖

> ⚠️ 虚拟环境下完成以下步骤。

* 在 `requirements.in` 中声明所有依赖：
    ```ini
    djangorestframework
    dj-rest-auth
    djangorestframework-simplejwt
    django-allauth # (dj-rest-auth 需要它来处理注册)
    ```
* 运行 `pip-compile` 和 `pip-sync`。

### 步骤 2: 配置 `settings.py` (核心)

这是在“配置乐高积木”的连接方式。

1.  **`INSTALLED_APPS` (注册积木):**
    * `rest_framework`：激活 DRF。
    * `rest_framework.authtoken`：(DRF 的老式 Token，但 `dj-rest-auth` 依赖它)。
    * `django.contrib.sites`：`allauth` 依赖它。
    * `allauth`, `allauth.account`, ...：`allauth` 的组件。
    * `dj_rest_auth`, `dj_rest_auth.registration`：`dj-rest-auth` 的组件。

    ```python
    INSTALLED_APPS = [
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
        # --- (1) 新增 - DRF 和 Auth 框架 ---
        'rest_framework',
        'rest_framework.authtoken',
        'dj_rest_auth',

        # 'dj_rest_auth' 需要 'sites' 框架
        'django.contrib.sites', 
        'allauth',
        'allauth.account',
        'allauth.socialaccount',
        'dj_rest_auth.registration',

        # 创建的App
        "phrase_log",
    ]
    ```

2.  **`MIDDLEWARE` (添加“门卫”):**
    * `allauth.account.middleware.AccountMiddleware`：这是 `allauth` 要求的一个“中间件”（门卫），用来处理请求。

    ```python
    MIDDLEWARE = [
        'django.middleware.security.SecurityMiddleware',
        'django.contrib.sessions.middleware.SessionMiddleware',
        'django.middleware.common.CommonMiddleware',
        'django.middleware.csrf.CsrfViewMiddleware',
        'django.contrib.auth.middleware.AuthenticationMiddleware',
        'django.contrib.messages.middleware.MessageMiddleware',
        'django.middleware.clickjacking.XFrameOptionsMiddleware',

        # --- 【ADD THIS LINE】 ---
        'allauth.account.middleware.AccountMiddleware',
    ]
    ```

3.  **`SITE_ID = 1` (必需品):**
    * `allauth` 和 `sites` 框架需要知道网站的 ID。`1` 是默认值。

4.  **`REST_FRAMEWORK = {...}` (配置 DRF):**
    * **这是需要理解的重点。**
    * `DEFAULT_AUTHENTICATION_CLASSES`: 我们告诉 DRF：“我们的默认认证方式是 `JWTAuthentication`。”（使用 `simplejwt`）
    * `DEFAULT_PERMISSION_CLASSES`: 我们告诉 DRF：“我们的默认权限是 `IsAuthenticated`。” (即：**所有 API 默认都是“拒绝访问”的**，除非你带了有效的 Token)。

5.  **`ACCOUNT_...` (配置 allauth):**
    * **这是需要理解的重点。**
    * `ACCOUNT_AUTHENTICATION_METHOD = 'email'`：我们把登录方式从“用户名”改成了“Email”，这更现代。
    * `ACCOUNT_EMAIL_VERIFICATION = 'none'`：我们**暂时关闭**了“注册后必须验证邮箱”的功能，否则在本地开发时你收不到邮件，永远无法登录。

### 步骤 3: 暴露 URL 并迁移

1.  **`urls.py` (挂上“门牌”):**
    * 我们使用 `path('api/auth/', include('dj_rest-auth.urls'))` 把 `dj-rest-auth` 提供的**所有** API 接口（如 `login`, `logout`...）自动挂载到了 `/api/auth/` 路径下。

2.  **`python manage.py migrate` (施工):**
    * **这是需要理解的重点。**
    * 我们**没有**运行 `makemigrations`，因为我们没有*修改*我们自己的 `models.py`。
    * 我们只是在 `migrate`（施工），执行那些 `allauth` 和 `sites` 包**自带**的“蓝图”（迁移文件），在数据库中创建它们需要的表。


### 步骤 4：

1. 创建超级管理员。

2. 启动服务器。

3. 登录 Admmin 后台。

4. 修改 `Sites` Table 中唯一一行的数据，把 `Domain name` 改成 `localhost:8000`。
---

## 3. 结论：我需要理解什么？

不需要“背诵” `settings.py` 里的每一行配置。

但**必须理解**我们这套架构的**“分工”**：

* **需要理解：** 为什么我们需要 4 个包，而不是 1 个？（因为它们各自负责 API、认证逻辑、JWT令牌）。
* **需要理解：** `REST_FRAMEWORK` 里的配置是什么意思？（它在设置全局的“认证”和“权限”规则）。
* **需要理解：** `ACCOUNT_AUTHENTICATION_METHOD = 'email'` 这种配置的作用是什么？（它在定义网站的*业务逻辑*）。
* **需要理解：** `migrate` 和 `makemigrations` 的区别。