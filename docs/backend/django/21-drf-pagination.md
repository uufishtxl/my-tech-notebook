# DRF 分页策略

#django #drf #pagination

## 为什么需要分页

数据量大时，一次性返回所有数据会导致：
- 响应时间过长
- 内存压力
- 前端渲染卡顿

## 前后端分工

### 后端职责

```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 20
}
```

**API 响应结构**：

```json
{
    "count": 150,
    "next": "http://api/logs/?page=2",
    "previous": null,
    "results": [...]
}
```

### 前端职责

1. 初始请求第一页
2. 提供分页 UI 控件
3. 根据 `next`/`previous` 链接获取其他页

## 排序

```python
queryset = PhraseLog.objects.all().order_by('-created_at')
#                                          ↑ 减号表示降序
```

| 写法 | 排序方式 |
|------|----------|
| `order_by('created_at')` | 升序（最早优先） |
| `order_by('-created_at')` | **降序**（最新优先） |
