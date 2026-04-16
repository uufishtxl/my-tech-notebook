#django #drf #serializer #optimization

在 DRF 中，关联字段（ForeignKey）默认只返回 ID（如 `"source_audio": 12`）。但在实际业务中，前端往往需要更多信息。

我们有两种策略：**轻型（扁平化）** 和 **重型（结构化）**。

---

## 策略一：轻型狙击（Flat / Lightweight）

**场景**：你只需要关联对象的**某一个或两个字段**（比如只需要书名，不需要作者的生日、国籍等）。

**核心技巧**：`source` 参数 + 点号链式查找。

Python

```
class AudioChunkSerializer(serializers.ModelSerializer):
    # 【核心】无需嵌套，直接"隔空取物"
    # source='source_audio.title' 告诉 DRF：去 source_audio 对象里找 title 属性
    title = serializers.CharField(
        source='source_audio.title', 
        read_only=True
    )
    
    class Meta:
        model = AudioChunk
        # 结果：title 会平级地出现在 JSON 根节点，没有多余的大括号
        fields = ['id', 'chunk_index', 'title']
```

**JSON 效果：**

JSON

```
{
  "id": 1,
  "chunk_index": 5,
  "title": "Python入门教程"  // <--- 扁平化，直接就在这
}
```

---

## 策略二：重型轰炸（Nested / Heavyweight）

**场景**：你需要关联对象的**完整详情**（比如需要显示作者的头像、姓名、简介等全套信息）。

**核心技巧**：**Serializer 嵌套**（外包给另一个翻译官）。

Python

```
class SliceSerializer(serializers.ModelSerializer):
    # 【核心】实例化另一个 Serializer
    # 必须加 read_only=True，否则写操作（POST/PUT）时 DRF 会期待你传一个大字典
    audio_file = AudioFileSerializer(read_only=True)

    class Meta:
        model = Slice
        fields = ['id', 'start_time', 'audio_file']
```

**JSON 效果：**

JSON

```
{
  "id": 1,
  "start_time": 10.5,
  "audio_file": {            // <--- 结构化，是一个对象
      "id": 55,
      "url": "http://...",
      "duration": 120
  }
}
```

---

## ⚔️ 战术对比：什么时候用哪个？

|**特性**|**策略一：轻型 (source)**|**策略二：重型 (Nested Serializer)**|
|---|---|---|
|**数据结构**|**扁平 (Flat)**|**嵌套 (Tree/Dict)**|
|**代码量**|极少（一行字段定义）|较多（需要定义另一个 Serializer 类）|
|**前端体验**|取值方便 `data.title`|需要深层取值 `data.audio_file.url`|
|**适用场景**|只需要 **1-2 个字段** 用于列表展示|需要 **完整对象** 用于详情页展示|
|**读写性质**|纯只读 (Derived Field)|通常用于只读展示 (Read Only)|

---

## 💡 性能优化必读 (N+1 问题)

无论用哪种策略，只要跨了外键，DRF 内部在序列化循环时，都会去数据库查一次关联表。

**如果不优化，100 条数据就会产生 101 次 SQL 查询！**

**解决方案**：在 View (视图层) 必须配合 `select_related`。

Python

```
# views.py
class AudioChunkViewSet(viewsets.ModelViewSet):
    serializer_class = AudioChunkSerializer
    # 关键：提前把外键 source_audio 连表查出来，放入内存
    queryset = AudioChunk.objects.all().select_related('source_audio')
```

> [!TIP]
> 
> **记住口诀**：
> 
> 想拿**一个词**，用 `source` 指过去。
> 
> 想拿**一本书**，把 `Serializer` 嵌进去。