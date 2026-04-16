# Django 模型设计踩坑实录与最佳实践

> [!NOTE]
> 本文总结了在设计 Django Model 时的常见细节陷阱，尤其是 `null` vs `blank`，JSONField 的容错处理，以及极其重要的数据库索引策略。

---

## 1. `null=True` vs `blank=True` 的终极抉择

- **字符串字段** (`CharField`, `TextField`)：
  建议**只用 `blank=True`**。如果加了 `null=True`，数据库里就可能出现空字符串 `""` 和 `NULL` 两种表示空的值，这会让后续的查询（如 `isnull` 和 `Exact=''`）变得非常头疼。
- **非字符串字段** (`ForeignKey`, `DateTimeField`, `IntegerField`, `JSONField`等)：
  如果允许为空，必须**两者同时使用**，即 `null=True, blank=True`。

## 2. 心法修炼：JSONField 的容错与防御性编程

在定义如 `SAnnotation` 中的 `ai_response` 等信息经常缺失或未生成的 `JSONField` 时：

### 2.1 `default=dict` 的防御性力量
- **原理**：强制字段在没有任何数据时返回 `{}` 而不是 `None`。
- **好处**：
    - **避坑 `AttributeError`**：你可以放心地写 `data.get('key')` 或 `for k, v in data.items()`。如果字段是 `None`，这些字典操作会直接导致程序崩溃；如果是 `{}`，则只是返回空或不进入循环。
    - **代码清爽**：省去了大量 `if data is not None:` 的非空判断。

### 2.2 为什么有的项目不设置它？ (状态语义化)
部分工程中用 `None` 表示“还没开始处理”，用 `{}` 表示“处理完了但是没拿回有效数据”。但这种基于数据类型来判断状态的做法容易出错，更靠谱的做法是专门用一个 `status` 字段（如 `PROCESSING`, `SUCCESS`）来追踪。因此，推荐依然加上 `default=dict`。

## 3. Other Model Tips
- **`updated_at` 的自动化**：使用 `auto_now=True`，每次 `.save()` 时自动刷新时间戳。
- **`Meta` 类语法**：必须使用赋值符号 `ordering = ['-created_at']`，千万别顺手写成字典的冒号 `:`。
- **`related_name`**：总是为了反向查询设置直观的复数名称，例如在使用外键连接 `Paragraph` 和 `Article` 时设置 `related_name='paragraphs'`，便于通过 `article.paragraphs.all()` 调用。

---

## 4. 数据库性能优化：复合索引篇

在开发长文章的段落表 (`Paragraph`) 时，遇到了性能优化的关键点。

### 4.1 索引是什么？
把数据库想象成一本没有目录的大部头。
- **没索引**：如果你要找“第 5 段”，数据库必须从头扫描。这就是 **全表扫描 (Full Table Scan)**。
- **有索引**：就像书后面的“关键词索引”，它记录了数据和物理位置的映射。查找复杂度从 $O(N)$ 降到了 $O(\log N)$。
- **代价**：占用额外的磁盘空间，并且每次新增/更新数据时都要修改索引表，稍微拖慢写性能。

### 4.2 为什么单列索引不如复合索引？ (`Meta.indexes`)
- **误区**：给所有看起来常用的字段单独加 `db_index=True`。
- **场景**：我们常常需要查询“**某篇文章**里的**所有段落 (按照 index 排序)**”。
- **分析**：
    1.  如果只给 `article_id` 加索引：数据库能很快找到这篇文章，但在拿到这一百个段落后，还得在内存里慢慢排顺。
    2.  如果只给 `index` 加索引：毫无意义，因为全库有无数个“第一段”。
    3.  **复合索引**：让数据库一步到位，直接定位到该文章的起始，并且按照 `index` 的顺序把数据切出来给你。

**正确写法：**
```python
class Paragraph(models.Model):
    article = models.ForeignKey(Article...)
    index = models.IntegerField(...)

    class Meta:
        indexes = [
            # 基于业务查询场景的复合索引
            models.Index(fields=['article', 'index']),
        ]
```

> [!TIP]
> 🚨 深度策略：当你为一个字段在 `Meta.indexes` 中建立了复合索引后，**不再需要**在那个单独的模型字段上标明 `db_index=True` 了。

---
*整理自：Reader 模块挑战实录*