* Django 框架

## ⭐️⭐️⭐️ `Model.objects.get()`和`Model.objects.filter()`有什么区别？

`get()`只用于获取唯一一条记录，如果找不到（`DoesNotExist`）或找到多条（`MultipleObjectsReturned`）都会抛出异常。`filter()`总是返回一个`QuerySet`（一个列表），即使里面是 0 个、1 个或多个对象。


## ⭐️⭐️⭐️ Django 的 `forms.Form` 和 `forms.ModelForm`有什么区别？

* `ModelForm` 和数据库模型 (`Model`) 强绑定，用于“表单字段和数据库字段一一对应”的场景（如注册、发帖），它有 `.save()` 方法。
* `forms.Form` 是通用的，不和 `Model` 绑定，用于“收集数据”的场景，比如“登录表单”、“搜索表单”，收集到的数据是用来调用 API 的，而不是直接存库。

## ⭐️⭐️⭐️ 用户提交表单成功后，为什么要用 `redirect`，而不是 `render`

这涉及到 POST/Redirect/GET (PRG) 模式，主要是为了防止表单重复提交。

如果 POST 成功后用 `render`返回页面：
* 用户刷新浏览器时，会重新提交之前的 POST 请求，导致数据重复创建
* 浏览器会显示警告“确认重新提交表单”

如果用 `redirect`
* 第一次 POST 处理成功后，返回302重定向到一个 GET 请求的页面
* 即使用户刷新，也只是刷新这个 GET 页面，不会重新提交表单
* 这是 Web 开发的最佳实践。

## ⭐️⭐️ `{% csrf_token %}`是什么？它解决了什么问题？

它是跨站请求伪造（CSRF，Cross-Site Request Forgery）令牌。作为一种恶意攻击，CSRF 会诱使已经登录了目标网站的用户，去访问一个恶意链接或页面，从而在用户不知情的情况下，以他的身份和权限执行一些非本意的操作，比如修改密码、转账等。

## ⭐️⭐️ `blank=True` 和 `null=True`在 `Model`字段中有什么区别？

* `blank=True` 是验证层的，允许表单（如 Admin 后台）提交空值。
* `null=True`是数据库层的，允许数据列存储

对于 `TextField` 或 `CharField`，官方推荐用 `blank=True`，而不是 `null=True`。

## ⭐️ 什么是 PRG 模式？为什么要用它？

`Post-Redirect-Get`。它解决了“`POST` 请求后刷新页面，浏览器会警告‘是否重新提交表单’”的问题。可以通过 `redirect()` 到 `results_view`，把一个 `POST` 请求转换成了一个安全的 `GET` 请求。

## Django 的 Session 是如何工作的？数据存在客户端还是服务器？

数据存在服务器（默认在 `django_session` 数据库表里）。客户端浏览器只存一个包含 `sessionid` 的 Cookie 作为“钥匙”。

## `pk__in` 是什么意思？

这是 Django ORM 的字段查询语法，会被翻译成 SQL 的 `WHERE id IN (...)` 语句。