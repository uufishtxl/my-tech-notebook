# Django ForeignKey 值的两种形态

#django #models #orm #foreign-key

## 核心概念

`ForeignKey` 字段在不同上下文中有两种不同的表现形式：

| 上下文 | 字段名示例 | 值类型 |
|--------|------------|--------|
| **Python 代码** | `source_audio` | 完整的 `SourceAudio` 对象实例 |
| **数据库** | `source_audio_id` | 整数（`SourceAudio` 的主键 ID） |

## Python/Django 代码中

```python
# 访问 ForeignKey 属性，返回完整的关联对象
my_chunk.source_audio  # → SourceAudio 对象

# 可以链式访问属性
my_chunk.source_audio.drama.name  # → "老友记"
```

## 数据库中

Django 自动创建带 `_id` 后缀的列：

```sql
-- audio_chunk 表结构
source_audio_id INTEGER  -- 存储的是 SourceAudio 的主键
```

> [!TIP]
> 如果只需要 ID 而不需要完整对象，可以直接访问 `my_chunk.source_audio_id`，这样不会触发额外的数据库查询。

## 相关链接

- [[08-django-models|Django Models 基础]]
- [[10-orm-m2m-relations|ORM 多对多关系]]

*记录于 2025-11-11*
