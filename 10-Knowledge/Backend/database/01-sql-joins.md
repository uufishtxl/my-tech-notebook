# SQL JOINs 图解指南

> [!NOTE]
> 很多开发者因为依赖 ORM (如 Django)，容易忽略 SQL JOIN 的直观含义。掌握它是理解数据库查询和解决性能问题的基础。
> **一句话总结**：JOIN 就是把两张表“横向”拼接在一起，区别在于“以谁为主”。

---

## 1. 核心概念可视

假设有两张表：
- **User (用户表)** (Left)
- **Order (订单表)** (Right)

### 1.1 INNER JOIN (内连接) -> "交集"
> 只保留**两边都有**的数据。

- **场景**：查询“下过单的用户”。
- **结果**：没下过单的用户不显示，没有所属用户的订单（脏数据）不显示。
- **Django**: `User.objects.filter(order__isnull=False)` (Implicit join)

### 1.2 LEFT JOIN (左连接) -> "保左" (最常用) 🌟
> **左边这张表全都要**。右边有匹配的就拼上去，没匹配的就填 NULL。

- **场景**：查询“所有用户及其订单详情（如果有的话）”。
- **结果**：
    - 下过单的用户 -> 显示订单信息。
    - **没下过单的用户 -> 依然显示在列表里**，但订单字段是空的 (NULL)。
- **关键点**：这是做“报表”或“列表页”最常用的 join，因为你通常不想因为用户没数据就把他漏掉。

### 1.3 RIGHT JOIN (右连接) -> "保右" (极少用)
> **右边这张表全都要**。左边有匹配的就拼上去，没匹配的就填 NULL。

- **场景**：几乎没有必须用 Right Join 的场景。
- **替代方案**：通常把表的位置换一下 (`FROM Order LEFT JOIN User`)，就变成 Left Join 了。符合人类从左到右的阅读习惯。

---

## 2. 为什么面试官喜欢问？

1.  **数据完整性意识**：用 IP (Inner Join) 可能会意外“丢数据”（漏掉没关联记录的主体）。用 Left Join 更安全。
2.  **性能陷阱**：如果不分青红皂白全是 Left Join，可能会在右表数据量巨大时变慢。

---

## 3. Django ORM 对应关系

在 Django 中，你很少直接写 JOIN，但你需要知道 ORM 在干什么：

| SQL 概念 | Django ORM 写法 |
| :--- | :--- |
| **INNER JOIN** | `Model.objects.filter(related_field__isnull=False)` |
| **LEFT JOIN** | **默认的** `select_related()` 就是 Left Outer Join。 <br> 或者查询主表时，反向查询子表 (OneToMany)。 |
| **RIGHT JOIN** | Django ORM 原生不支持直接写 Right Join (因为它可以用 Left Join 替代)。 |

---

## 4. 经验总结

> [!TIP]
> **初学者口诀**：
> - **INNER**: 两情相悦才在一起 (交集)。
> - **LEFT**: 我(左)全都要，你有就给，没有拉倒。
> - **RIGHT**: (基本不用，忘了吧)。

*创建日期: 2026-01-28*
