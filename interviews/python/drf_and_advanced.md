# DRF 与后端进阶

## ⭐️⭐️⭐️⭐️⭐️ 如何实现API的数据权限隔离，让不同用户只能看到自己的数据？

这是一个考察数据权限控制的经典问题，核心是覆写 `ViewSet` 中的 `get_queryset()` 方法。

默认情况下，`ViewSet` 的 `queryset` 属性会获取模型的所有对象。为了实现数据隔离，我们需要根据发出请求的用户 (`request.user`) 来动态地过滤查询集。

**回答范例：**
“通过覆写 `get_queryset` 方法，我们可以访问到 `self.request.user`，并用它来过滤查询集。例如，`return Drama.objects.filter(user=self.request.user)`。这样，无论是列表查询 (`.list`) 还是详情查询 (`.retrieve`)，数据源从一开始就是经过权限过滤的，确保了用户只能访问授权给他们的数据。”

```python
class DramaViewSet(viewsets.ReadOnlyModelViewSet):
	serializer_class = DramaSerializer
	permission_classes = [IsAuthenticated]
	
	def get_queryset(self):
		# 核心：根据当前请求的用户来过滤数据
		return Drama.objects.filter(user=self.request.user)
```

## ⭐️⭐️⭐️⭐️⭐️ 在创建资源时，如何安全地将当前登录用户关联上？

这是一个考察对安全和后端逻辑理解的常见问题。绝对不能让前端在 POST 请求体中直接传递 `user_id`，因为这是不安全的。正确的做法是在后端自动注入当前登录的用户信息。

核心是覆写 `ViewSet` 中的 `perform_create()` 方法。

**回答范例：**
“首先，在 `Serializer` 中将 `user` 字段设为 `read_only=True`，防止它被客户端提交的数据修改。然后，在 `ViewSet` 中覆写 `perform_create` 方法，这个方法是 `.create()` 动作的一个内部钩子。在调用 `serializer.save()` 时，将 `request.user` 作为额外参数传入，例如 `serializer.save(user=self.request.user)`。这样就由后端安全地完成了用户关联。”

## ⭐️⭐️⭐️⭐️ 请描述一下DRF中从用户请求到获取到`request.user`的完整认证流程。

这是一个考察对认证机制深入理解的问题。能清晰描述这个流程会非常加分。

**回答范例：**
“这个流程大致如下：
1.  **前端发送请求**：在请求头 (Header) 的 `Authorization` 字段中携带 `Bearer <access_token>`。
2.  **Django中间件**：请求到达 Django 后，会经过一系列中间件，其中 `AuthenticationMiddleware` 会被触发。
3.  **DRF认证类接管**：`AuthenticationMiddleware` 会调用在 `settings.py` 中配置的 DRF 认证类（比如 `JWTAuthentication`）。
4.  **Token验证与解析**：`JWTAuthentication` 类会从请求头中提取 Token，用后端的 `SECRET_KEY` 验证其签名。如果验证通过，就从 Token 中解析出 `user_id`。
5.  **查询用户**：用解析出的 `user_id` 去数据库中查询，获取到完整的、可信的 `User` 模型对象。
6.  **挂载到Request**：最后，认证类将这个 `User` 对象挂载到 `request` 对象上，成为 `request.user`。
这样，在后续的视图逻辑中，我们就可以通过 `request.user` 安全地访问到当前登录的用户了。”

## ⭐️⭐️⭐️⭐️ 如果一个API需要处理耗时任务，如何设计才能避免用户长时间等待？

这是一个考察系统设计和异步编程思想的问题，能体现候选人对性能和用户体验的考量。

**回答范例：**
“对于耗时任务（如视频转码、批量数据处理），不能在 API 视图中同步执行，否则会导致请求超时和糟糕的用户体验。正确的做法是引入异步任务队列，比如 `Celery`。

工作流程是：
1.  **立即响应**：API 视图接收到请求后，不直接执行任务，而是将任务和参数发送到消息队列（如 Redis 或 RabbitMQ），然后立即返回一个 `202 Accepted` 响应给用户，告知请求已被接收。
2.  **后台处理**：一个或多个独立的 `Celery` worker 进程在后台监听消息队列。当发现新任务时，它们会获取任务并异步执行。
3.  **结果通知（可选）**：任务完成后，可以通过 WebSocket、回调 URL 或状态查询接口等方式通知用户结果。

这种‘任务外包’的模式，将 API 的快速响应和耗时的后台处理解耦，是处理此类问题的最佳实践。”

## ⭐️⭐️⭐️ 为什么创建多对多关系时，需要先`.save()`再`.set()`?

这是一个考察对 ORM 和数据库关系理解的经典问题。

**回答范例：**
“因为多对多关系是通过一个独立的‘中间表’来实现的。这个中间表的每一行都记录了两个关联模型的主键（ID）。

当我们在内存中创建一个新的模型实例时（例如 `my_instance = MyModel(...)`），它还没有被存入数据库，因此它没有主键（ID）。

如果我们不先 `.save()` 就直接尝试 `.set()` 多对多关系，Django 就不知道应该在中间表的 `mymodel_id` 字段中填入什么值。

所以，必须先调用 `.save()` 将主对象存入数据库以获得一个确定的主键，然后才能调用 `.set()` 或 `.add()` 在中间表中创建关联记录。”

## ⭐️⭐️⭐️ 如何为ViewSet添加一个非标准的、自定义的API接口？

这是一个考察 DRF 扩展能力的问题。

**回答范例：**
“通过使用 `@action` 装饰器。`@action` 可以让我们在 `ViewSet` 中定义额外的路由。

- 通过 `detail=True` 或 `detail=False` 参数，可以决定这个动作是作用于单个资源实例（如 `/resource/{pk}/my_action/`）还是作用于资源集合（如 `/resource/my_action/`）。
- 通过 `methods` 参数可以指定支持的 HTTP 方法，如 `['get']` 或 `['post']`。

这对于实现一些非 CRUD 的功能，比如批量操作、触发某个特定流程等非常有用。”

## ⭐️⭐️⭐️ 解释一下Python的`with`语句和上下文管理器是如何工作的。

这是一个考察 Python 语言基础和设计模式的问题。

**回答范例：**
“`with` 语句是为了简化 `try...finally` 这种资源管理模式而设计的。它能确保无论代码块是否发生异常，都能执行必要的清理操作（如关闭文件、释放锁）。

它的工作依赖于‘上下文管理协议’，任何遵守这个协议的对象都可以被 `with` 使用。这个协议包含两个核心的魔法方法：
- `__enter__(self)`：在进入 `with` 代码块时被调用。它的返回值会赋给 `as` 后面的变量。
- `__exit__(self, exc_type, exc_val, exc_tb)`：在退出 `with` 代码块时被调用，无论成功还是失败。如果代码块中发生了异常，异常的类型、值和追溯信息会作为参数传给它，方便进行异常处理。所有的清理逻辑都写在这个方法里。”

## ⭐️⭐️ DRF序列化器中`many=True`的作用是什么？在什么场景下使用？

**回答范例：**
“`many=True` 是告诉序列化器，期望处理的是一个对象列表，而不是单个对象。

主要用在两个场景：
1.  **序列化（读取）**：当需要序列化一个查询集（QuerySet）或一个对象列表时，必须在创建序列化器实例时传入 `many=True`，例如 `MySerializer(my_queryset, many=True)`。
2.  **反序列化（写入）**：当需要验证和反序列化一个包含多个对象的 JSON 数组时（比如批量创建或批量更新），也必须传入 `many=True`，例如 `serializer = self.get_serializer(data=request.data, many=True)`。此时，`.is_valid()` 会对列表中的每一项进行验证。”

## ⭐️⭐️ 什么是Django Signals？请举一个使用场景。

这是一个考察应用解耦设计的问题。

**回答范例：**
“Django Signals 是一套内置的发布/订阅系统，它允许应用中的不同部分在发生特定事件时进行通信，而无需相互之间有直接的依赖。

一个经典的使用场景是：当一个模型被保存后，自动执行某些操作。

例如，我希望在每次创建一个新的 `SourceAudio` 模型后，自动触发一个‘预切片’的后台任务。我可以定义一个函数，并用 `@receiver(post_save, sender=SourceAudio)` 装饰器将它注册为 `post_save` 信号的接收器。这样，每当 `SourceAudio` 实例被成功保存后，Django 就会自动调用这个函数，而我无需在视图或模型代码中硬编码这个调用，实现了逻辑的解耦。”
