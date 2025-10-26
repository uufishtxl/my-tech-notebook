# 01. Django 模板语法 (DTL) 核心

Django 模板语言 (Django Template Language, DTL) 是一种用于在 Django 视图 (Views) 和前端 HTML 之间架起桥梁的标记语言。它允许开发者将后端数据动态嵌入到 HTML 页面中，并执行基本的逻辑控制。

DTL 的语法主要由三部分组成：

1.  **变量 (Variables):** `{{ variable }}`，用于输出值。
2.  **标签 (Tags):** `{% tag %}`，用于执行逻辑或控制流。
3.  **过滤器 (Filters):** `{{ variable|filter }}`，用于修改待显示的变量。

## 核心语法对比：`{{ }}` vs `{% %}`

| 语法 | 名称 | 用途 | 示例 |
| :--- | :--- | :--- | :--- |
| `{{ ... }}` | 变量 (Variable) | 从上下文中渲染并输出数据。 | `{{ post.title }}` |
| `{% ... %}` | 标签 (Tag) | 提供模板逻辑，如循环、条件判断、继承等。 | `{% if user.is_authenticated %}` |
| `{# ... #}` | 注释 (Comment) | 用于模板内的注释，不会在最终的 HTML 中渲染。 | `{# 这是一个模板注释 #}` |

---

## 1. 变量 (Variables): `{{ ... }}`

变量用于将视图传递过来的上下文 (context) 数据显示在页面上。Django 模板引擎会自动处理不同数据类型的访问。

* **访问对象属性:** `{{ my_object.attribute_name }}`
* **访问字典键值:** `{{ my_dict.key }}`
* **访问列表索引:** `{{ my_list.0 }}`
* **调用对象方法:** `{{ user.get_full_name }}` (注意：不能传递参数)

### 示例

```html
<h1>{{ post.title }}</h1>
<p>作者: {{ post.author.username }}</p>
<p>第一个标签: {{ tags.0 }}</p>
```

## 2. 标签（Tags）：`{% ... %}`

标签是 DTL 的“动力引擎”，用于控制模板的渲染流程和逻辑。

### A. 模板继承 (Template Inheritance)

这是构建可维护网站的基础，允许你定义一个基础框架（“骨架”），其他页面可以继承和填充它。

- `{% extends 'base.html' %}`

  - 声明当前模板继承自 base.html。
  - 规则: 必须是模板文件中的第一个标签。

- `{% block content %} ... {% endblock %}`

  - 在父模板中定义一个可被子模板覆盖的“块”（插槽）。
  - 子模板使用相同的 {% block %} 标签来填充内容。

### 示例

`base.html` (父模板)

```html
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}默认标题{% endblock %}</title>
</head>
<body>
    <header>
        {% include 'partials/_header.html' %}
    </header>
    
    <main>
        {% block content %}
        {% endblock %}
    </main>
</body>
</html>
```

`child.html` (子模板)

```html
{% extends 'base.html' %}

{% block title %}
    我的页面标题
{% endblock %}

{% block content %}
    <h1>这是子页面的具体内容</h1>
    <p>...</p>
{% endblock %}
```

### B. 逻辑与控制流

`{% if %}` / `{% elif %}` / `{% else %}`

用于条件判断。支持 `and`, `or`, `not` 以及 `==`, `!=`, `<`, `>`, `<=`, `>=` 等操作符。

```html
{% if user.is_authenticated %}
    <p>欢迎, {{ user.username }}!</p>
{% elif user.is_guest %}
    <p>欢迎, 游客!</p>
{% else %}
    <p>请 <a href="{% url 'login' %}">登录</a>.</p>
{% endif %}
```

`{% for %}`

用于遍历可迭代对象（如列表、QuerySet）。

- `{% empty %}`: 当迭代的列表为空时，将执行 {% empty %} 块内的代码。

- `forloop` 变量: 在循环内部，DTL 提供了一些内置变量：

  - `forloop.counter`: 循环计数 (从 1 开始)

  - `forloop.counter0`: 循环计数 (从 0 开始)

  - `forloop.first`: 是否为第一次循环 (True/False)

  - `forloop.last`: 是否为最后一次循环 (True/False)

```html
<ul>
{% for item in item_list %}
    <li class="{% if forloop.first %}first{% endif %}">
        {{ forloop.counter }}. {{ item.name }}
    </li>
{% empty %}
    <li>没有可显示的项目。</li>
{% endfor %}
</ul>
```

### C. 加载与包含 (Loading & Inclusion)

`{% load static %}` 和 `{% static %}`

`{% load static %}` 标签用于加载 Django 的 `static` 标签库，之后才能使用 `{% static %}` 标签来生成静态文件（CSS, JS, 图片）的 URL。

- 作用: 动态管理静态文件路径，便于部署时统一配置。

- 位置: 通常放在 `{% extends %}` 标签之后，文件的顶部。

```html
{% load static %}
<link rel="stylesheet" href="{% static 'css/style.css' %}">
<img src="{% static 'images/logo.png' %}" alt="Logo">
···

`{% include %}`

用于将另一个模板文件的内容插入到当前位置。非常适合用于重构可复用的 HTML 片段（如导航栏、页脚）。

```html
<body>
    {% include 'partials/_navigation.html' %}
    
    <div class="content">
        ...
    </div>
    
    {% include 'partials/_footer.html' %}
</body>
```

### D. 其他关键标签

`{% url %}`

(极其重要) 用于动态地根据 urls.py 中定义的 URL 模式的 `name` 来反向生成 URL 路径。这避免了在模板中硬编码 URL。

```python
# urls.py 示例:
# path('', views.home, name='homepage')
# path('posts/<int:post_id>/', views.post_detail, name='post_detail')
```

```html
<a href="{% url 'homepage' %}">返回首页</a>
<a href="{% url 'post_detail' post.id %}">{{ post.title }}</a>
```

`{% csrf_token %}`

```html
<form method="POST" action="">
    {% csrf_token %}
    <button type="submit">提交</button>
</form>
```

## 过滤器 (Filters): `{{ ... | ... }}`

过滤器用于在变量显示之前对其进行修改或格式化。

语法: `{{ variable | filter_name }}` 或 `{{ variable | filter_name:"argument" }}`

### 常用过滤器

| 过滤器 | 描述 | 示例 |
| :--- | :--- | :--- |
| `date` | 格式化日期时间对象。 | `{{ post.created_at \| date:"Y-m-d H:i" }}` |
| `truncatewords` | 将字符串截断为指定数量的单词。 | `{{ post.body \| truncatewords:30 }}` |
| `length` | 返回变量的长度（适用于列表、字符串等）。 | `{{ my_list \| length }}` |
| `upper` / `lower` | 转换为大写或小写。 | `{{ "Hello" \| lower }}` |
| `default` | 如果变量为 `False` 或 `None`，则使用默认值。 | `{{ user.bio \| default:"该用户很懒..." }}` |
| `safe` | (慎用) 标记字符串为“安全”，使其不被 Django 转义。 | `{{ post.html_content \| safe }}` |