# 页面二：模型与数据库 (Models & DB)

## 1. 定义模型 (Models)

Model 是数据的单一、权威的来源。

* 位置：`<app_name>/models.py`
* 规则：继承 `django.db.models.Model`

```python
from django.db import models

class MenuItem(models.Model):
    name = models.CharField(max_length=255)
    description = models.TextField(blank=True)
    price = models.DecimalField(max_digits=6, decimal_places=2)
    # ... 更多字段

    def __str__(self):
        return self.name
```

## 2. 数据库迁移 (Migrations)

迁移是 Django 将你的模型更改（如添加字段、删除模型）同步到数据库 Schema 的方式。

1. 生成迁移文件：

  - `python manage.py makemigrations`
  - 这会检查 `models.py` 的变动，并生成一个 `migrations/` 目录下的新文件。

2. 执行迁移：

  - `python manage.py migrate`
  - 这将 `migrations/` 文件中的 `SQL` 指令应用到数据库。

## 3. 配置数据库

### 默认数据库 (SQLite)

* Django 默认使用 SQLite。
* 配置在 `settings.py` 的 `DATABASES` 字典中，默认指向项目根目录的 `db.sqlite3` 文件。

### 更改为 MySQL

1. 准备数据库：

   * 下载并运行 MySQL。
   * 登录: `mysql -u root -p`
   * 创建数据库: `CREATE DATABASE <db_name> CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci`

   **其他命令：**

   * `use <db_name>;`：
   * `show tables;`

2. 安装驱动：

   * `pip install mysqlclient`

3. 修改 `settings.py`：

   * 修改 `DATABASES` 字典中的 `default` 条目。
   * 推荐： 使用环境变量管理敏感信息（你提到了 `env` 文件）。

```python
# settings.py
import os
from decouple import config # 推荐使用 python-decouple

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': config('DB_NAME'),         # 'NAME': '<db_name>',
        'USER': config('DB_USER'),         # 'USER': 'root',
        'PASSWORD': config('DB_PASSWORD'), # 'PASSWORD': 'your_password',
        'HOST': config('DB_HOST', default='127.0.0.1'),
        'PORT': config('DB_PORT', default='3306'),
        'OPTIONS': {
            'init_command': "SET sql_mode='STRICT_TRANS_TABLES'",
        }
    }
}
```

