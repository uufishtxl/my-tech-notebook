
## Why Django Shell
我想利用既有项目（lingua-workbench）已经完整的虚拟环境来巩固一些 Django 知识点。那么无意能在这个项目中创建一个 `sandbox` App 来测试是最方便的。而我们可以利用 Shell 快速创建一些假数据 (Seeds)。 

## 创建 `sandbox` App

为了更好地理解 Django 的相关知识，可以在项目中建立一个沙盘 App，来进行一些测试。

1. 创建 `sandbox` App。
	```bash
	python manage.py startapp sandbox
	```
2. 将 `sandbox` 加入到 `settings.py` 中的 `INSTALLED_APPS` 中。
```python
# settings.py

INSTALLED_APPS = [
    # ...
    'rest_framework',
	# ...    
    'sandbox',  # <--- 加上这一行，练功房正式开张！
]
```
3. 将 `sandbox` 的 `urls` 添加到项目的 `urls.py` 文件中。
```python
# sample_project/urls.py

from django.urls import path, include  # 记得导入 include

urlpatterns = [
    # ... 原有的 admin 等
    
    # ✅ 加上这行，给你的练功房开个门
    # 以后访问 http://127.0.0.1:8000/sandbox/Xxx 就会进到这里
    path('sandbox/', include('sandbox.urls')), 
]
```
4. 进行数据库迁移。
```bash
python manage.py makemigrations sandbox
python manage.py migrate sandbox
```

## 使用 Django Shell

1. 将 `ipython` 安装到虚拟环境。
2. 运行 Shell。
```bash
python manage.py shell
```


### 创建种子数据

安装了 `ipython` 后，可以直接将多行代码复制到 Shell 中。
```python
from sandbox.models import Author, Book

# 1. 造 5 个作者
for i in range(5):
    author = Author.objects.create(name=f"Author_{i}", age=30+i)
    
    # 2. 每个作者写 3 本书
    for j in range(3):
        Book.objects.create(title=f"Book_{i}_{j}", author=author)

print(f"现在有 {Author.objects.count()} 个作者，{Book.objects.count()} 本书")
```
### 编写 `for` 循环并运行

IPython 需要通过一个**空行**来判断你已经完成了整个代码块（Block）的输入。因此在完成编写后，只要多按一次 **ENTER** 就可以。

### 如何在 `Shell` 中使用代码提示

除了直接填充整行，你还可以根据需要选择以下操作：

|**快捷键**|**功能**|**说明**|
|---|---|---|
|**`→` (右方向键)**|**接受全部提示**|将光标移动到行尾，并填充所有灰色代码。|
|**`End`**|**接受全部提示**|效果同右方向键，将光标跳到行尾并填充。|
|**`Ctrl` + `E`**|**接受全部提示**|这是 Emacs 风格的快捷键，效果同上。|
|**`Ctrl` + `→`**|**逐词接受 (Partial)**|**非常实用！** 如果你只想要灰色提示的前半部分（例如只要 `print` 不要后面的括号），按住 Ctrl 再按右箭头，可以一个单词一个单词地填充。|
