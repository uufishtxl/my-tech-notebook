# Django Modesl

## 创建外键

```Python
from django.db import models
from django.conf import settings

# 假设这个字段在一个叫 Post 的 Model 内部
class Post(models.Model):
    # ...
    user = models.ForeignKey(
        settings.AUTH_USER_MODEL, 
        on_delete=models.CASCADE,
        related_name='posts' # 推荐添加，用于反向查询
    )
```

**定义**：用于创建**一对多**（one-to-many）关系。它将当前 Model (如 `Post`) 关联到另一个 Model (如 `User`)。

**核心参数解析**：

- `settings.AUTH_USER_MODEL`：指向关联的模型。这是引用 Django 内置用户模型的**最佳实践**，而不是直接硬编码 `auth.User`。
    
- `on_delete=models.CASCADE`：**关键策略**。它定义了当**被引用的对象**（即 `User`）被删除时，**包含此外键的对象**（即 `Post`）应该如何处理。
    
    - `CASCADE` (级联删除)：表示如果某个用户被删除了，那么该用户所有的 `Post` 也会被一并删除。
        
- `related_name='posts'`：(推荐) 允许从 `User` 对象反向查询。例如：`some_user.posts.all()`。

## 正整数字段（`PositiveIntegerField`)

```python
from django.db import models

class QuizAttempt(models.Model):
    # ...
    failed = models.PositiveIntegerField(
        default=0, 
        help_text="Count of times this was failed in a quiz"
    )
```

**定义**：一个整数字段，但数据库层面会增加约束，确保其值必须为**正数或零**（non-negative）。

**常用参数解析**：

- `default=0`：设置新记录被创建时，如果未提供此字段的值，则默认为 `0`。对于 `null=False` 的字段（`PositiveIntegerField` 默认 `null=False`），提供 `default` 是个好习惯。
    
- `help_text="..."`：在 Django Admin 管理后台的表单中，显示在此字段下方的辅助提示文字。
    
## 多对多字段 (ManyToManyField)

```python
from django.db import models

# 假设我们还有一个 Tag Model
# class Tag(models.Model):
#     name = models.CharField(max_length=100)

class Article(models.Model):
    # ...
    tags = models.ManyToManyField(
        'Tag',                  # 1. 关联的模型（推荐用字符串）
        blank=True,             # 2. 在Admin中允许为空
        related_name='articles' # 3. 用于反向查询
    )
```

**定义**：用于创建**多对多**（many-to-many）关系。例如，一篇 `Article` 可以有多个 `Tag`，同时一个 `Tag` 也可以被用于多篇 `Article`。

**工作原理**：Django 会自动在数据库中为您创建一个**中间表**（junction table 或 "through" table）来管理这种关联关系。您通常不需要直接操作这个中间表。

**核心参数解析**：

1. `'Tag'` (第一个参数)：指向相关联的 Model。使用字符串（`'Appname.ModelName'` 或 `'ModelName'`）可以避免循环导入（circular import）的问题。
    
2. `blank=True`：在 Django Admin 或表单验证时，允许此字段为空（即一篇文章可以不关联任何标签）。对于多对多关系，这通常是推荐设置。
    
3. `related_name='articles'`：**非常重要**。它允许您从 `Tag` 对象反向查询。例如，获取某个标签下的所有文章：`some_tag.articles.all()`。