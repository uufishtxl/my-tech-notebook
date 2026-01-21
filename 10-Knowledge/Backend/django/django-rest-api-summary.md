# Django REST Framework API 快速指南

本文档根据日常笔记整理，旨在总结使用 Django REST Framework (DRF) 创建 API 的核心概念和最佳实践。

## 一、API 开发基础

### 1. 创建 GET API 的三步流程

创建一个只读的 API 端点（Endpoint）通常遵循以下三个步骤：

1.  **创建 Serializer**：定义如何将 Django 的 Model 实例或其他复杂数据类型转换为可以轻松渲染成 JSON 的原生 Python 数据类型。
2.  **创建 API View**：编写视图逻辑，用于处理 HTTP 请求。它会查询数据库，使用 Serializer 序列化数据，并返回 HTTP 响应。
3.  **注册 URL**：将 API View 关联到一个具体的 URL 路径，使其可以被外部访问。

### 2. Serializer (序列化器)

**是什么？**

Serializer（通常翻译为“序列化器”）是 DRF 的核心组件之一。它的主要工作是：

*   **序列化 (Serialization)**：将从数据库获取的 QuerySet 或 Model 实例等复杂数据，转换为 JSON、XML 等格式，以便通过网络传输给客户端。
*   **反序列化 (Deserialization)**：将客户端发送过来的数据（如 JSON）转换回 Django 的 Model 实例，通常用于数据验证和保存。

**关于 `many=True` 参数**

在实例化一个 Serializer 时，`many=True` 参数至关重要：

*   当你要序列化的目标是一个**列表**（如 Django 的 QuerySet 或多个 Model 实例的列表）时，必须设置 `many=True`。
*   当你只序列化**单个**对象实例时，应省略该参数或设置为 `many=False` (默认值)。

```python
# 序列化单个对象
serializer = PhraseLogSerializer(log_instance)

# 序列化对象列表 (QuerySet)
logs_queryset = PhraseLog.objects.all()
serializer = PhraseLogSerializer(logs_queryset, many=True)
```

> **关于术语翻译**:
> 在中文技术社区中，“Serializer”通常不翻译，直接使用英文。如果需要翻译，“序列化器”是比较准确的译法。“Serialize”则对应“序列化”。

### 3. 处理大量数据：分页 (Pagination)

当一个 API 端点可能返回大量数据时，一次性返回所有数据会造成服务器和客户端的性能问题。**分页**是解决此问题的标准策略。

*   **后端**：负责将数据分割成固定大小的“页”，并在 API 响应中提供元数据，如总记录数、总页数、下一页/上一页的链接等。
*   **前端**：根据后端返回的元数据，提供 UI 控件（如“加载更多”按钮或页码），让用户可以浏览不同的数据页。

**在 DRF 中实现分页**

最简单的方式是使用 DRF 内置的分页功能，并结合泛型视图 `ListAPIView`。

**步骤 1：在 `settings.py` 中配置全局分页样式**

```python
REST_FRAMEWORK = {
    # ... 其他设置 ...

    # --- 新增分页配置 ---
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10,  # 每页默认显示 10 条记录
}
```

**步骤 2：使用 `ListAPIView` 泛型视图**

`ListAPIView` 专门用于处理返回对象列表的 GET 请求，并会自动集成在 `settings.py` 中配置的分页功能。

```python
from rest_framework.generics import ListAPIView
from rest_framework.permissions import IsAuthenticated
from .models import PhraseLog
from .serializers import PhraseLogSerializer

class HistoryAPIView(ListAPIView):
    """
    处理“获取历史记录”的 GET 请求。
    继承自 ListAPIView 后，将自动拥有分页和列表查询功能。
    """
    permission_classes = [IsAuthenticated]
    serializer_class = PhraseLogSerializer  # 指定要使用的序列化器

    def get_queryset(self):
        """
        此方法定义了数据源。
        DRF 会自动对这个 QuerySet 进行分页处理。
        """
        # 返回当前已认证用户的所有记录，并按创建时间降序排列
        return PhraseLog.objects.filter(user=self.request.user).order_by('-created_at')
```

> **`APIView` vs `ListAPIView`**
>
> *   `APIView`：是一个基础视图，提供了处理 HTTP 请求（`get`, `post` 等）的基本功能，但你需要**手动**完成所有事情：查询数据、实例化 Serializer（并根据情况传入 `many=True`）、创建 `Response` 对象等。它像一辆“手动挡汽车”，给予你完全的控制。
> *   `ListAPIView`：是一个更高级的泛型视图，它继承自 `APIView` 并预设了处理列表数据的标准流程。你只需要告诉它 `serializer_class` 和 `queryset`（或实现 `get_queryset` 方法），它就会**自动**处理数据查询、序列化（自动判断 `many=True`）和分页。它像一辆“自动挡汽车”，极大简化了开发。

## 二、API 更新与修改

### 1. `PUT` vs `PATCH`

在设计更新资源的 API 时，`PUT` 和 `PATCH` 是两种常用的 HTTP 方法，但它们有本质区别：

*   **`PUT` (替换)**：要求客户端提供**完整**的资源表示。如果某个字段没有在请求中提供，`PUT` 可能会将其设置为空或默认值，相当于用新数据完全替换旧数据。
*   **`PATCH` (部分更新)**：只要求客户端提供需要**修改**的字段。未提供的字段将保持不变。

**实现 `PATCH` 的一般步骤**

```python
# 在一个继承自 APIView 的视图中
def patch(self, request, pk):
    try:
        instance = MyModel.objects.get(pk=pk)
    except MyModel.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    # 实例化 Serializer 时，传入三个关键参数：
    # 1. instance: 要更新的原始对象
    # 2. data: 包含新数据的 request.data
    # 3. partial=True: 告诉 Serializer 这是部分更新
    serializer = MySerializer(instance, data=request.data, partial=True)

    if serializer.is_valid():
        serializer.save()
        return Response(serializer.data)
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

### 2. 使用 `RetrieveUpdateDestroyAPIView` 简化操作

为了避免为单个对象的 GET、PUT、PATCH、DELETE 操作编写重复的代码，DRF 提供了 `RetrieveUpdateDestroyAPIView`。

这个泛型视图将所有这些功能集于一身。它通过 URL 中的查找字段（默认为 `pk`）来定位单个对象实例。

```python
from rest_framework.generics import RetrieveUpdateDestroyAPIView

class LogDetailAPIView(RetrieveUpdateDestroyAPIView):
    """
    处理单个日志记录的 GET (检索), PUT/PATCH (更新), DELETE (删除) 请求。
    """
    permission_classes = [IsAuthenticated]
    serializer_class = PhraseLogSerializer
    queryset = PhraseLog.objects.all() # 定义了可操作的数据范围

    # 如果需要更复杂的查询逻辑，可以重写 get_queryset
    # def get_queryset(self):
    #     return PhraseLog.objects.filter(user=self.request.user)
```

**URL 配置**

这种视图的 URL 通常包含一个参数，如 `<int:pk>`，用于标识要操作的对象。

```python
# urls.py
from django.urls import path
from .views import LogDetailAPIView

urlpatterns = [
    path('api/logs/<int:pk>/', LogDetailAPIView.as_view(), name='log-detail'),
]
```

## 三、常见问题解答

**1. `request.user` 是如何自动附加到请求中的？**

是的。当你为视图配置了认证类（如 `JWTAuthentication`）和权限类（如 `IsAuthenticated`）后，DRF 的中间件会在处理请求时执行以下操作：

1.  从请求头（如 `Authorization: Bearer <token>`）中解析出认证信息（如 JWT）。
2.  验证该信息的有效性。
3.  如果验证成功，它会从数据库中检索对应的用户对象。
4.  最后，将这个用户对象附加到 `request.user` 属性上。

因此，在你的视图逻辑中，可以直接通过 `self.request.user` 安全地访问当前已登录的用户，而无需手动处理认证逻辑。
