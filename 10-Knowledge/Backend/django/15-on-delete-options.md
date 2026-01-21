# Django Model `on_delete` 选项详解

#django #models #database #foreign-key

在 Django 中定义 `ForeignKey` 关系时，必须指定当被引用的"父"对象被删除时，关联的"子"对象应该如何处理。

## 选项一览

| 选项 | 行为 | 使用场景 |
|------|------|----------|
| `CASCADE` | 删除父对象时，子对象一并删除 | 最常用，如删除用户时删除其帖子 |
| `PROTECT` | 阻止删除，抛出 `ProtectedError` | 保护关键数据完整性 |
| `SET_NULL` | 设置外键为 `NULL` | 需要 `null=True` |
| `SET_DEFAULT` | 设置为默认值 | 需要设置 `default` |
| `DO_NOTHING` | 不执行任何操作 | ⚠️ 危险，可能导致数据库错误 |
| `RESTRICT` | 类似 PROTECT，抛出 `RestrictedError` | 更严格的保护 |

## PROTECT 详解

```python
drama = models.ForeignKey(Drama, on_delete=models.PROTECT)
```

> [!WARNING]
> 使用 `PROTECT` 时，无法删除仍有子对象关联的父对象。必须先删除或重新关联所有子对象。

**比喻**：就像一把安全锁——你不能拆掉地基（`Drama`），如果上面还有建筑物（`SourceAudio`）存在。

## 相关链接

- [[08-django-models|Django Models 基础]]

*记录于 2025-11-11*
