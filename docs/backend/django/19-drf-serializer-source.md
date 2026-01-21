# DRF Serializer 跨外键取字段

#django #drf #serializer

## 核心技巧：`source` 参数

无需数据库迁移，直接在 Serializer 中获取关联模型的字段：

```python
class AudioChunkSerializer(serializers.ModelSerializer):
    # 跨外键获取 SourceAudio 的 title
    title = serializers.CharField(
        source='source_audio.title',  # 点号链式查找
        read_only=True
    )
    
    class Meta:
        model = AudioChunk
        fields = ['id', 'source_audio', 'chunk_index', 'file', 'title']
```

## source 参数解析

`source='source_audio.title'` 告诉 DRF：
1. 找到 `AudioChunk` 的 `source_audio` 属性
2. 获取该 `SourceAudio` 对象的 `title`
3. 作为 API 响应中 `title` 字段的值

## 关键点

| 要点 | 说明 |
|------|------|
| `read_only=True` | 派生字段应设为只读 |
| **无需迁移** | 纯表现层改动，不影响数据库 |
| **减少请求** | 客户端无需额外请求关联数据 |

> [!TIP]
> 这是 API 设计的高效模式，提供"扁平化"数据结构给客户端。
