---
index: 13
gemini_id: a6f5a78b363f7707
synced_at: 2026-01-20T11:49:26.628Z
---

## Q

那django是天然支持 Foreign Key 在对象层面上去搜索关联表别的属性的值吗？

## A

是的，**Django 在这一点上比 SQLAlchemy 更“天然”及“自动化”。**

如果在 SQLAlchemy 里你需要手动配 `relationship` 和 `back_populates` 才能实现“双向导航”，那在 Django 里，**只要写了 `ForeignKey`，这一切都是自动赠送的**。

我们可以把 Django 的逻辑概括为：**“定义了物理外键，逻辑关系自动生成”。**

具体分两种情况（**访问**和**搜索**），我对比着刚才的 SQLAlchemy 给你讲：

### 1\. 对象层面的访问 (Object Access)

假设你在 Django 里只写了这么一行：

```
# Django models.py
class Order(models.Model):
    # 只要这一行！不用写 relationship
    customer = models.ForeignKey(Customer, on_delete=models.CASCADE) 

``` 

**场景 A：正向访问（拿着订单找人）**

*   **Django:** `my_order.customer.name`
    
*   **体验:** 和 SQLAlchemy 配置好 `relationship` 后完全一样。Django 会自动去帮你查关联表。
    

**场景 B：反向访问（拿着人找订单）—— 重点！**

*   **SQLAlchemy:** 你必须显式定义 `orders = relationship(..., back_populates=...)`，否则 `customer` 对象上根本没有 `orders` 这个属性。
    
*   **Django:** 你**什么都不用写**。Django 默认会在 `Customer` 对象上给你挂载一个 **`表名_set`** 的属性。
    
    *   代码: `my_customer.order_set.all()`
        
    *   这就是 Django 的“天然支持”。如果你不喜欢 `order_set` 这个名字，可以在 ForeignKey 里加个参数 `related_name="orders"`，那就变成 `my_customer.orders.all()` 了。
        

* * *

### 2\. 搜索/过滤层面的操作 (Query/Filter)

你问的“搜索关联表别的属性的值”，在 Django 里叫 **跨关系查询 (Lookups that span relationships)**。

这是 Django 最著名的\*\*“双下划线魔法” (`__`)\*\*。

**场景：我想找出所有“张三”下的订单。**

*   **SQLAlchemy (比较繁琐):** 你需要 Join：
    
    ```
    session.query(Order).join(Customer).filter(Customer.name == "张三")
    
    ```
    
*   **Django (极其丝滑):** 你不需要手动 Join，直接用 `__` 穿透过去：
    
    ```
    # 翻译：查找 customer 字段对应的表里的 name 字段等于 "张三" 的 Order
    Order.objects.filter(customer__name="张三")
    
    ```
    
    甚至可以无限套娃：
    
    ```
    # 查找“属于住在上海的客户的”订单
    Order.objects.filter(customer__address__city="Shanghai")
    
    ```
    

### 总结对比

特性

SQLAlchemy

Django

**定义外键**

`Column(ForeignKey(...))`

`models.ForeignKey(...)`

**正向导航** (order.customer)

需要额外定义 `relationship`

**天然支持** (自动有了)

**反向导航** (customer.orders)

需要定义 `relationship(back_populates)`

**天然支持** (默认为 `foo_set`，可用 `related_name` 改名)

**跨表搜索**

需要显式 `.join()`

**天然支持** (使用 `__` 双下划线)

导出到 Google 表格

**一句话结论：** Django 的设计哲学是“约定优于配置”。它默认假设你定义了外键就是想双向访问、就是想跨表查，所以它**不需要**像 SQLAlchemy 那样显式地配置 `relationship` 就能在对象层面直接搜、直接用。
