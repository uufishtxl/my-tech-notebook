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