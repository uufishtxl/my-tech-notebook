# DRF APIView 类型对比

#django #drf #api

## ListAPIView vs APIView

| 特性 | ListAPIView (自动档) | APIView (手动档) |
|------|---------------------|------------------|
| 数据获取 | 自动调用 `get_queryset()` | 手动查询 |
| Serializer | 自动实例化 | 手动实例化 |
| `many=True` | 自动检测 QuerySet | **必须手动指定** |
| 分页 | 自动处理 | 手动处理 |

### ListAPIView 示例

```python
class PhraseLogListView(ListAPIView):
    serializer_class = PhraseLogSerializer
    
    def get_queryset(self):
        return PhraseLog.objects.filter(user=self.request.user)
```

### APIView 示例

```python
class TagAPIView(APIView):
    def get(self, request):
        tags = Tag.objects.filter(user=request.user)
        serializer = TagSerializer(tags, many=True)  # 必须手动指定 many=True
        return Response(serializer.data)
```

---

## Serializer `many` 参数

```python
# 单个对象
serializer = PhraseLogSerializer(log)

# 列表/QuerySet
serializer = PhraseLogSerializer(logs, many=True)
```

---

## RetrieveUpdateDestroyAPIView

继承后自带 GET、PUT、PATCH、DELETE 功能：

```python
class PhraseLogDetailView(RetrieveUpdateDestroyAPIView):
    serializer_class = PhraseLogSerializer
    queryset = PhraseLog.objects.all()
    # URL: /api/logs/<id>/
```

---

## PATCH vs PUT

| 方法 | 行为 | 字段要求 |
|------|------|----------|
| `PUT` | 替换整个资源 | 必须包含**所有**字段 |
| `PATCH` | 部分更新 | 只需包含要修改的字段 |

```python
serializer = PhraseLogSerializer(
    instance,           # 原数据
    data=request.data,  # 新数据
    partial=True        # PATCH 时设为 True
)
```
