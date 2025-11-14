# Django ORM > 多对多关系中“先有ID，再建关联原则“

为什么要先 `save()`，再`.set(tags)`？

## 多对多关系和中间表

我们在 `AudioSlice` 模型中定义了 `tags = models.ManyToManyField(AudioTag, ...)`。在数据表层面，Django 会为这个多对多关系创建一个额外的“中间表”（也叫“连接表”）。这个表的作用就是记录 `AudioSlice`和 `AudioTag`之间的关联关系。

这个表中间至少有两列：`audioslice_id` 和 `audiotag_id`。表中的每一行（例如 `(101, 5)`）都代表 “`ID` 为 101 的 `AudioSlice`” 与 `ID` 为 5的 `AudioTag` 之间存在一个关联。

## 先有鸡还是先有蛋的问题

当我们执行 `audio_slice_instance = slice_chunk_to_slice(...)`时，我们只是在 Python 的内存中创建了一个 `AudioSlice` 对象。此时，这个对象在数据库中还不存在，因此它没有主键（ID）。

为了在中间表创建一条关联记录（比如 `(audioslice_id=???, audiotag_id=5)`，我们必须知道 `audioslice_id`是多少。

这也就是为什么，我们会需要先通过 `.save()`，执行一条 `INSERT` SQL 语句，之后再通过 `set(tags)`来创建新的关联记录。

```Python
# ...
# audio_slicer/views.py
audio_slice_instance.save() # Save the instance to the database
audio_slice_instance.tags.set(tags) # Set tags after saving
# ...
```
