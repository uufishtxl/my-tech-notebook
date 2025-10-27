# 页面三：视图、模板与路由 (Views, Templates, URLs)

MVT 架构的核心是 Views, Templates 和 URLs 的协同工作。

## 1. 视图 (Django Views)

* 位置：`<app_name>/views.py`
* 作用：接收 `request` 对象，返回 `HttpResponse` 对象。

### 函数式视图 (Function-Based View)

```python
from django.shortcuts import render
from django.http import HttpResponse
from .models import MenuItem

def menu_list(request):
    items = MenuItem.objects.all()
    # "context" 是传递给模板的数据
    context = {
        'items': items,
    }
    # render 会结合模板和上下文，返回一个 HttpResponse
    return render(request, 'app_name/menu_list.html', context)
```

### 类式视图 (Class-Based View, CBV)

```python
from django.views import View
from django.shortcuts import render
from .models import MenuItem

class MenuListView(View):
    def get(self, request):
        items = MenuItem.objects.all()
        context = {
            'items': items,
        }
        return render(request, 'app_name/menu_list.html', context)
```

## 2. 路由管理 (URLs)

URLs 负责将用户请求的 URL 路径分发给正确的 View 函数/类。

### 1. 项目级路由

* 位置：`<project_name>/urls.py`
* 作用：项目的“总路由表”，通常用 `include` 来分发到各个 App。

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    # 将所有 /menu/ 开头的 URL 都转交给 menu_app.urls 文件处理
    path('menu/', include('menu_app.urls')), 
]
```

### 2. App 级路由

* 位置：在 App 目录中手动创建 `urls.py` 文件。
* 作用：处理该 App 内部的具体路径。

```python
# <app_name>/urls.py
from django.urls import path
from . import views

urlpatterns = [
    # /menu/
    path('', views.MenuListView.as_view(), name='menu_list'),
    # /menu/item/5/
    # path('item/<int:pk>/', views.MenuItemView.as_view(), name='menu_item'),
]
```

* *注意： 类式视图必须调用 `.as_view()` 方法。*
* *`name='...'` 是一个好习惯，用于在模板中反向解析 URL。*

### 3. 模板（Django Templates）

* 位置：在 App 文件夹中创建 `templates/` 文件夹。
* 最佳实践： 在 `templates/` 内部再创建一个以 `<app_name>` 命名的文件夹（如：<app_name>/templates/<app_name>/），以避免 App 间的模板命名冲突。
* 示例：`menu_app/templates/menu_app/menu_list.html`

```html
<h1>菜单列表</h1>
<ul>
    {% for item in items %}
        <li>{{ item.name }} - ${{ item.price }}</li>
    {% empty %}
        <li>暂无菜单项。</li>
    {% endfor %}
</ul>
```

* [点击查看 Templates 更多常用语法](./90-templates-syntax.md)


