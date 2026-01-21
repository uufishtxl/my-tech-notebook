# 页面四：表单与后台管理 (Forms & Admin)

## 1. Django 表单 (Forms)

Form 负责从用户那里接收数据，并验证数据是否符合预期。

### 1. 创建 Form

* 在 App 目录中**手动创建** `forms.py` 文件。
* 推荐使用 `ModelForm`，它可以直接从 Model 生成表单字段。

```python
# <app_name>/forms.py
from django import forms
from .models import SampleModel

class SampleForm(forms.ModelForm):
    class Meta:
        model = SampleModel
        fields = '__all__' # 或者 ['field1', 'field2']
```

### 2. 在模板中渲染 Form

```html
<form method="POST">
    {% csrf_token %}
    
    {{ form.as_p }}
    
    <button type="submit">保存</button>
</form>
```

* `{{ form }}`: 键名 form 必须在 View 的 context 字典中提供。
* `{% csrf_token %}`的作用：防止 CSRF 攻击。

### 3. 在 View 中处理 Form (POST 请求)

View 需要同时处理 `GET` (显示表单) 和 `POST` (处理提交的数据)。

```python
# <app_name>/views.py
from django.shortcuts import render, redirect
from .forms import SampleForm

def sample_form_view(request):
    # 如果是 POST 请求 (用户提交了表单)
    if request.method == 'POST':
        # 1. 获取 Form 中填写的数据
        form = SampleForm(request.POST)
        
        # 2. 验证 Form 数据是否符合 Schema
        if form.is_valid():
            # 3. 保存数据到 Model 中
            form.save()
            # 4. 重定向到一个成功页面 (PRG 模式)
            return redirect('success_page_name') 
            
    # 如果是 GET 请求 (用户刚打开页面)
    else:
        form = SampleForm() # 创建一个空表单

    # 渲染页面，传入 form 实例
    return render(request, 'app_name/form_page.html', {'form': form})
```

## 2. Django 后台管理 (Admin)

### 1. 创建超级管理员

* `python manage.py createsuperuser`
* 按照提示设置用户名、邮箱和密码。

### 2. 访问后台

* 启动服务器：`python manage.py runserver`
* 访问：`http://127.0.0.1:8000/admin`
* 使用你刚创建的用户名和密码登录。

### 3. 向后台注册 Model

* 默认情况下，后台是空的。你必须“注册”你的 Model，才能在后台管理它。
* 位置：<app_name>/admin.py

```python
# <app_name>/admin.py
from django.contrib import admin
from .models import MenuItem, SampleModel

# 注册 Model
admin.site.register(MenuItem)
admin.site.register(SampleModel)
```

* 注册后，你就可以在 admin 页面通过表单对 Model 进行数据填充（增删改查）。