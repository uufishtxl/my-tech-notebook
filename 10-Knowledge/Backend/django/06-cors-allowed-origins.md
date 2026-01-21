# 跨源资源共享

当涉及非同源的资源需要进行共享时，需要在后端设置允许的跨域访问列表。

## 步骤一：安装依赖

在 `requirements.in`中添加 Django 的跨源资源共享支持依赖包，并且运行 `pip-compile requirements.in` 与 `pip-sync`安装依赖。

## 步骤二：在 `INSTALLED_APPS` 中列出该 APP

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'rest_framework.authtoken',
    'dj_rest_auth',
    'django.contrib.sites', 
    'allauth',
    'allauth.account',
    'allauth.socialaccount',
    'dj_rest_auth.registration',
    # --- 在这里列出 ---
    'corsheaders',
    'phrase_log',
]
```

## 步骤三：在 MIDDLEWARE 中列出

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
        # --- 【ADD THIS LINE】；必须位于 CommonMiddleware前面 ---
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'allauth.account.middleware.AccountMiddleware',
]
```

## 步骤四：定义 CORS 允许访问列表

如果想要在开发时更宽松一点，可以这样设置：

```Python
CORS_ALLOW_ALL_ORIGINS = True
```

但是，上线后必须换成这样的列表设置方式：

```Python
# 位于列表底部
CORS_ALLOWED_ORIGINS = [
    # Vue CLI (npm run dev) 默认的开发服务器地址
    "http://localhost:5173", 
    "http://127.0.0.1:5173",
]
```

## 步骤五：凭证

默认情况下，CORS 请求不会发送“凭证”（比如浏览器的`Cookies`或`Authorization`头）。当前端需要发送“凭证”（比如`dj-rest-auth`的`JWT`认证 Cookie）给后端时，必须在后端明确说明“允许跨域请求携带凭证”。

```Python
# 告诉浏览器，我们允许“带凭证”(Cookies, Auth headers)的跨域请求
CORS_ALLOW_CREDENTIALS = True
```