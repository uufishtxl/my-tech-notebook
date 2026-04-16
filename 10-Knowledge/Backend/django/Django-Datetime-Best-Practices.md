# Django 时间与日期处理最佳实践

> [!NOTE]
> 在处理“时间敏感型”应用（如番茄钟、打卡系统）时，遵循“存储用 UTC，显示用本地”的基本原则。

---

## 1. 核心工具集

### 1.1 绝对禁忌：不要使用 `datetime.now()`
- **理由**：它是“时区盲” (Naive)，在启用 `USE_TZ = True` 的 Django 中会导致逻辑冲突和时间误差。
- **正解**：`from django.utils import timezone; now = timezone.now()`。它返回的是一个带时区信息的实（Aware）对象。

### 1.2 时间加减：`timedelta`
用于处理“过去 24 小时”、“剩余时间”等偏移逻辑：
```python
from datetime import timedelta
from django.utils import timezone

start_window = timezone.now() - timedelta(days=7) # 获取一周前的时间点
```

---

## 2. ORM 查询技巧：手术刀级的精确过滤

### 2.1 时间截断检索 `__date`
- **痛点**：数据库存的是 `DateTimeField` (包含秒)，但前端只想按“天”查。
- **方案**：`Pomodoro.objects.filter(created_at__date='2026-02-28')`。数据库会在底层自动执行截断匹配。

### 2.2 常用衍生符
- `__year='2026'`
- `__month='11'`
- `__week_day` (注意数值索引：1=周日, 2=周一... 依次类推)

---

## 3. `auto_now_add` 的逻辑陷阱
- **`auto_now_add=True`**：仅在对象**第一次创建**时设置（记录生日）。
- **`auto_now=True`**：每次调用 `.save()` 发生修改时都会更新（追踪更新时间）。
- **避坑指南**：像 `completed_at` (完成时间) 这种依赖业务逻辑手动触发的字段，**不能用上述参数**，必须设为 `null=True, blank=True`，在 `perform_update` 钩子中手工通过 `timezone.now()` 填入。

---

*创建日期: 2026-02-28*
