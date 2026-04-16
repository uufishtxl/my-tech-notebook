# Django ORM 实战速查表 (Quick Review)

## 🏎️ 性能之王：.update() vs .save()
在处理多个模型实例时，选对更新方式能让响应速度提升 10 倍以上：

### 1. 传统逐个保存模式 (Slow)
- **做法**：先 `get()` 或遍历 `filter()` 出来的每一个对象，修改属性后 `save()`。
- **代价**：假设有 100 个词要更新，数据库会收到 **100 条 SQL 指令**。
- **风险**：非原子性，中间报错可能导致部分入库。

### 2. 原子批量更新模式 (Fast) 🌟
- **做法**：直接在 `filter()` 结果后面挂 `.update(field=value)`。
- **收益**：单一 SQL 指令（WHERE IN），性能极高，自带数据库原子性。

---

## 🧠 F() 表达式：在数据库里做加法
- **原理**：直接引用数据库字段原值（如：`box_level = F('box_level') + 1`）。
- **收益**：有效防止“竞态条件”（Race Condition），避免 Python 逻辑覆盖掉数据库已有的更新。

## 🔍 Q() 对象：逻辑拼装大师
构建复杂的 `WHERE` 子句（OR, NOT, 嵌套）：
- **逻辑“或” (|)**: `Q(a=1) | Q(b=2)`
- **逻辑“非” (~)**: `~Q(status='PENDING')`
- **逻辑“与” (&)**: `Q(a=1) & Q(b=2)`

---

## 🌌 跨次元的计算冲突：Django F() 对象的边界
当你尝试在 `.update()` 中结合 Python 指令（如 `timedelta` 或 `**`）与 `F()` 时，会发生**环境错位**：

### 1. 为什么 `timedelta(days=F('...'))` 会报错？
- **Python 环境 (timedelta)**: 需要在代码执行那一刻拿到定值。
- **数据库环境 (F)**: 只代表数据库格子的占位符。
- **结论**: Python 算不出“现在 + 还没读出来的天数”。

### 2. 为什么 `2 ** F('box_level')` 无法运行？
- **操作符错位**: `**` 是 Python 符号，无法翻译成跨数据库通用的 SQL 函数（如 `POWER()`）。

---

## 🛑 QuerySet 的“坑与禁区”

### 1. 切片锁 (QuerySet Slicing)
- **SQL 对等**: `queryset[:10]` -> `LIMIT 10`。
- **禁忌**: 一旦执行切片，QuerySet 即刻被“锁定/求值”，不能再追加 `.filter()` 或 `.order_by()`。

### 2. 随机排序的代价 (order_by('?'))
- **后果**: 大表下会导致全表扫描，性能崩塌。仅限微量数据使用。

### 3. "获取或创建" (get_or_create)
- **返回结构**: 元组 `(object, created)`。务必注意解构。

---

## 🕒 性能优化：字段拉取方式 (.values)
不要拉取全表字段，大幅提升查询速度：
- **字典列表 (.values)**: `[{'id': 1}, ...]` 适合 JSON 序列化。
- **元组列表 (.values_list)**: 内存占用极低。
- **单列扁平化 (flat=True)**: `[1, 2, 3]` 构建 ID 列表的必杀技。

---

## 🔥 失踪的更新日期：update_fields 的副作用
这是一个关于 `save(update_fields=...)` 的严重陷阱：

- **现象**: 你更新了 `status`，但 `updated_at` (auto_now=True) **竟然没有动**。
- **原理**: 使用 `update_fields` 后，Django 会严格按照这份“排他清单”生成 SQL。既然你没写 `updated_at`，它就绝对不回偷偷帮你更新，自动刷新的钩子（Hook）会直接被跳过。
- **避坑法**: 每次调用局部保存时，请务必养成连带写上 `updated_at` 的习惯：
  `obj.save(update_fields=['field_name', 'updated_at'])`

---
Created by Antigravity AI助手