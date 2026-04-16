# DRF ViewSet 生命周期与高级技巧

> [!NOTE]
> 深入理解 ModelViewSet 的底层运行机制，包括 Request 挂载、方法签名以及 QuerySet 的安全复用原则。

---

## 1. 核心运行机制

### 1.1 `self.request` 的挂载（APIView 层次）
当请求进入 ViewSet 并触发 `dispatch()` 时，DRF 已经将原生 `HttpRequest` 包装为 DRF `Request` 并直接**挂载为 `self.request`**。
- **意义**：在 `get_queryset`, `get_serializer` 等内置方法中可随时随地通过 `self.request` 获取用户信息、Params 等，无需手动传参。

### 1.2 `@action` 自定义方法的签名
所有绑定的 HTTP 动作方法（除 `self`）都必须**显式**接收 `request` 作为第一个位置参数：
```python
@action(detail=False, methods=['get'])
def ongoing(self, request):
    # request 与 self.request 指向同一对象
    # 必须手动返回 Response() 对象
    return Response(serializer.data)
```

### 1.3 `Response` 返回原则
- **ViewSet 内置方法 (`list`, `retrieve`)**：框架底层自动处理 `Response` 封装。
- **自定义 `@action`**：作为逻辑终点，必须手工返回 `Response`，否则 Django 中间件报错。
- **生命周期钩子 (`perform_create`, `perform_update`)**：属于内部干活的“插槽”，**不得返回 Response**。

---

## 2. 最佳实践：QuerySet 的复用与隔离

### 2.1 什么时候用 `self.get_queryset()`？
- **场景**：GET 渲染类动作（如 `list`, `history`）。
- **优势**：继承了已有的安全过滤（如限定当前用户）和 URL 级动态筛选功能，代码极致 DRY。

### 2.2 什么时候严禁使用 `self.get_queryset()`？
- **场景**：POST/PUT/PATCH 入库前的“防重/冲突”检查嗅探。
- **原因**：防止请求参数（Query Params）污染检查范围。
- **正确做法**：使用纯净的 `Model.objects.filter(...)` 从原始表基座开始查。

---

*创建日期: 2026-02-28*
