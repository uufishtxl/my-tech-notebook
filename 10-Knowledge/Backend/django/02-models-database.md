# 页面二：模型与数据库 (Models & DB)

## 1. 定义模型 (Models)

Model 是数据的单一、权威的来源。

* 位置：`<app_name>/models.py`
* 规则：继承 `django.db.models.Model`

```python
from django.db import models

class MenuItem(models.Model):
    name = models.CharField(max_length=255)
    description = models.TextField(blank=True) # blank=True 表示允许这个字段为空；null=True 则表示允许数据库列存储 NULL 值，Django 官方不推荐使用 null=True
    price = models.DecimalField(max_digits=6, decimal_places=2) # DecimalField 必须用于存储金钱 (price)、汇率或任何不能有浮点数（FloatField）舍入误差的数字。max_digits=6 这个数字总共允许的最大位数。这包括了小数点前和小数点后的所有位数。decimal_places=2 小数点后固定存储的位数（也叫“标度”）。
    # ... 更多字段

    def __str__(self):
        return self.name
```

## 2. 数据库迁移 (Migrations)

迁移是 Django 将你的模型更改（如添加字段、删除模型）同步到数据库 Schema 的方式。

1. 生成迁移文件：

   * `python manage.py makemigrations`
   * 这会检查 `models.py` 的变动，并生成一个 `migrations/` 目录下的新文件。

2. 执行迁移：

   * `python manage.py migrate`
   * 这将 `migrations/` 文件中的 `SQL` 指令应用到数据库。

## 3. 配置数据库

### 默认数据库 (SQLite)

* Django 默认使用 SQLite。
* 配置在 `settings.py` 的 `DATABASES` 字典中，默认指向项目根目录的 `db.sqlite3` 文件。

### 更改为 MySQL

1. 准备数据库：

   * 下载并运行 MySQL。
   * 登录: `mysql -u root -p`
   * 创建数据库: `CREATE DATABASE <db_name> CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci`
     * `CHARACTER SET utf8mb4`
       * `CHARACTER SET`：定义数据库用什么“字符集”来存储文本
       * `utf8mb4`：这是 MySQL 对 `utf8` 的一种“增强”实现。`mb4` 代表 "4-byte Multi-Byte"。
       * 为什么重要： 传统的 utf8 在 MySQL 中只支持3个字节，这导致它无法存储 4 字节的字符，最著名的例子就是 Emoji (表情符号 😄)。utf8mb4 修复了这个问题，是现代 Web 开发的事实标准。
     * `COLLATE utf8mb4_unicode_ci`
       * `COLLATE`：定义数据库用什么“排序规则”来比较和排序文本。
       * `utf8mb4_unicode_ci`：
         * `utf8mb4`: 告诉它这是用于 `utf8mb4` 字符集的。
         * `unicode`: 使用 Unicode 标准进行排序（这能保证跨语言排序的准确性）。
         * `ci` (这是关键): 代表 Case Insensitive (不区分大小写)。

    `_ci`（不区分大小写）的后果：

    * 排序时： "apple", "Banana", "cherry" 会被正确排序。
    * 查询时： `WHERE name = 'gemini'` 这个查询也会找到 `name = 'Gemini'` 或 `name = 'GEMINI'` 的记录。

    这条 SQL 命令的意思是：“创建一个数据库，它必须能正确存储 Emoji (用 `utf8mb4`)，并且在排序或搜索文本时不要区分大小写 (用 `_ci`)。”

   **其他命令：**

   * `use <db_name>;`：
   * `show tables;`

2. 安装驱动：

   * `pip install mysqlclient`

3. 修改 `settings.py`：

   * 修改 `DATABASES` 字典中的 `default` 条目。
   * 推荐： 使用环境变量管理敏感信息（`env` 文件）。

`env` 文件

```Ini, TOML
# .env
# 这是你的敏感信息文件。
# 警告：这个文件绝对、必须、一定要加入到 .gitignore 中！

# Django
SECRET_KEY=akjsdfl8-asdfjkl_this-is-not-real-askjdfh
DEBUG=True

# 数据库
DB_NAME=my_django_db
DB_USER=root
DB_PASSWORD=my_secret_password
DB_HOST=127.0.0.1
DB_PORT=3306
```

`.gitignore`

```
# .gitignore
...
.env  <-- 加上这行
venv/
...
```

在 `settings.py` 中引入 decouple，使用它的 `config`方法灵活取得敏感配置信息。

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

