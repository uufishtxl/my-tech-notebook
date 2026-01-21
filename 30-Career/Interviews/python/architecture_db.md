# 软件架构与数据库

** ⭐️⭐️⭐️ 谈一谈 Django 和 SQLAlchemy 的 ORM 工作流程和事务机制

两者的核心区别在于事务管理和现象状态跟踪机制的不同：

* SQLAlchemy采用工作单元模式，需要显示控制：
  * `session.add()`：将对象加入会话的待跟踪列表
  * `session.flush()`：将挂起的操作发送到数据库，生成 ID 但未提交事务
  * `session.commit()`：最终提交事务

* Django ORM 采用更简单的主动记录模式：
  * `Model.objects.create()`：直接执行 INSERT 并提交
  * 返回的对象立即拥有数据库生成的 ID

这种设计差异导致了：
* SQLAlchemy：需要手动 flush 才能获取自增 ID，但支持函更复杂的事务操作
* Django ORM：使用更简单，但在复杂事务场景下控制力不足

## ⭐️⭐️⭐️ SQLAlchemy 和 Django 如何批量 `INSERT` 数据

### Django

Django 提供了简单直接的 `bulk_create()`，适合大多数场景，但牺牲了灵活性和部分ORM特性。SQLAlchemy 则提供了从简单到极致的多种方案：从 ORM 的 `add_all()` 到 Core 层的原生批量插入，让开发者可以在便利性和性能之间做精确权衡。

**一般情形**

```python
# 一次性创建多个对象
objects = [MyModel(name=f'name_{i}') for i in range(1000)]
MyModel.objects.bulk_create(objects)  # 单次SQL执行
```
**极端性能场景**

```python
from django.db import connection
with connection.cursor() as cursor:
    cursor.executemany(
        "INSERT INTO myapp_mymodel (name) VALUES (%s)",
        [(f'name_{i}',) for i in range(1000)]
    )
```

### SQLAlchemy

**核心偏高插入（最高性能）**

```python
from sqlalchemy import insert
# 单次INSERT多值
stmt = insert(MyModel.__table__).values([{'name': f'name_{i}'} for i in range(1000)])
session.execute(stmt)
session.commit()
```

**ORM 批量插入**

```python
# 方式A：逐个add后统一commit
objects = [MyModel(name=f'name_{i}') for i in range(1000)]
session.add_all(objects)
session.commit()  # 这里会生成批量INSERT

# 方式B：bulk_save_objects (类似Django的bulk_create)
session.bulk_save_objects(objects)
session.commit()
```



## ⭐️⭐️ 解释下 MVC、MVP 和 MVVM 模式的区别

都是为了解耦。MVC 是经典模式，但 V 和 M 有耦合。MVP 用 Presenter 彻底解耦了 V 和 M，但 P 很“臃肿”。MVVM (如 Vue) 用“数据绑定”技术自动化了 MVP 的手动更新，效率最高。

### 更细化的理解

* MVC
  * 用户操作交给 Controller，由它更新 Model，再通知 View 刷新。
  * View 需要知道 Model 的结构来渲染界面，这导致了二者的耦合
* MVP
  * 用 Presenter 作为中间人，彻底切断了 View 和 Model 的直接联系。
  * View 变得“愚蠢”，只负责显示 UI 和转发用户输入
  * Presenter 持有 View 和 Model 的引用，承担了所有协调的工作。
  * 遗留问题：手动同步，Presenter 需要编写大量代码来更新 View，导致它变得臃肿。
* MVVM
  * 引入 ViewModel 和数据绑定
  * ViewModel 是 Model 的抽象，暴露了 View 需要的状态和命令。
  * 数据绑定引擎自动同步 View 和 View Model 的状态，无需手动更新。

## ⭐️ 对比下 Django 和 FastAPI

Django 是“全家桶” (Opinionated)，自带 ORM, Admin, Forms，适合快速开发内容管理系统。FastAPI 是“乐高积木” (Unopinionated)，只负责 API 和数据验证 (Pydantic)，性能极高，为异步而生，需要自己组合 ORM（如 SQLAlchemy）。

## ⭐️ 为什么要用 `utf8mb4`，而不是 `utf8`

因为 MySQL 的 `utf8` 是个“假”的，只支持3字节，存不了 Emoji (表情符号)。`utf8mb4` 才是真正的 UTF-8，支持4字节。

