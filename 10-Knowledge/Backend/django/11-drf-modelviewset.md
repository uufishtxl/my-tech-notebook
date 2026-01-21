# DRF > viewsets.ModelViewSet

## `ModelViewSet`

**描述**：是一个高度封装的类，它自动为模型提供了一套标准的、符合 RESTful 风格的 CRUD API 接口，提供基础的 CRUD 行为。

当创建一个 ViewSet 并继承自 `viewsets.ModelViewSet`时，DRF 会自动生成以下6个核心的 API “动作”（actions），并由 `Router`自动映射到对应的 URL 和 HTTP 方法：

| 动作 (Action)         | HTTP 方法  | URL 路径            | 行为描述                                                   |
| :------------------ | :------- | :---------------- | :----------------------------------------------------- |
| `.list()`           | `GET`    | `/resource/`      | **查询 (列表)**: 返回资源对象的列表，通常是分页的。                         |
| `.retrieve()`       | `GET`    | `/resource/{pk}/` | **查询 (单个)**: 根据主键 `pk` 返回单个资源对象的详细信息。                  |
| `.create()`         | `POST`   | `/resource/`      | **创建**: 接收请求数据，验证后创建一个新的资源对象。                          |
| `.update()`         | `PUT`    | `/resource/{pk}/` | **更新 (整体)**: 接收请求数据，对指定 `pk` 的对象进行**完整**更新（所有字段都需要提供）。 |
| `.partial_update()` | `PATCH`  | `/resource/{pk}/` | **更新 (部分)**: 接收请求数据，对指定 `pk` 的对象进行**部分**更新（只更新提供的字段）。  |
| `.destroy()`        | `DELETE` | `/resource/{pk}/` | **删除**: 根据主键 `pk` 删除一个资源对象。                            |

**基础配置**：对于任何继承自 `GenericViewSet`（包括  `ReadOnlyModelViewSet`）并使用了`ListModelMixin` 或  `RetrieveModelMixin` 的 `ViewSet`，至少需要定义 `serializer_class`。除了 `serializer_class`，通常还需要定义 `queryset`属性（或者覆盖 `get_queryset()`方法），告诉 DRF 这个 `ViewSet`应该操作哪些模型实例。没有 `queryset`，DRF 就不知道要列出或检索什么数据。

## 覆写的方法和场景总结

### `perform_create`覆写：在创建时注入额外数据

* **默认行为**：`.create()`动作会调用 `serializer.save()`，直接保存从请求中获取的数据。
* **自定需求**：在创建 `SourceAudio`对象时，希望这个对象与当前登录的用户关联起来，但我们**不希望**前端在请求中手动发送 `user_id`（因为这不安全）
* **覆盖方式**：
	* 在 `Serializer` 中明确将 `user`标记为 `read_only`，这就是告诉 `Serializer` `user`字段是输出字段，不期望从客户端接收输入，也不需要对其进行验证。
	* 在 `ViewSet` 中覆盖 `.perform_create`方法。这个方法是 `.create()`内部的一个钩子。通过在 `serializer.save()`中传入 `user=self.request.user`，确保 `user`字段由后端安全地设置。
> 这里可以注意辨别`request.user` 和 `request.data`的区别。
> * `request.data`是前端发送的请求体中的数据
> * `request.user`是由后端中间件和认证类“塞”到 `request`对象里的。 

#### 一个 API 请求在 Django 后端的全过程：
* 前端发送请求，`Header` 中会包含 `Authorization`字段，内容是 `Bearer <access_token>`
* 请求到达 Django，进入 `Viewset` 代码前，会先穿过一系列的中间件。
* `AuthenticationMiddleware`开始工作，这是其中一个关键的中间件，它的工作是“检查这个请求是谁发送的”。
* DRF 的 `JWTAuthentication`接管：
	* `AuthenticationMiddleware`会调用在 `settings.py`中配置的 DRF 认证类，在 lingua-workbench项目中是 `JWTAuthentication`
	* `JWTAuthentication`会专门在请求头里寻找 `Authorization`字段，取得 JWT 令牌字符串。
- `JWTAuthentication`使用只有后端知道的 SECRET_KEY 来解码和验证这个令牌的签名，确保它没有被伪造和篡改如果令牌有效，就从令牌中解析出用户的信息，通常是 `user_id`。之后用这个 `user_id`去数据库中查询，获取到完整的，真实的 `User`模型对象。
- 最后，`JWTAuthentication`将这个从数据库中查出来的、可信的 `User`对象，作为一个属性附加到 `request`对象上，这个属性的名字就叫 `user`。

```Mermaid
flowchart TD
    A[前端发送请求] --> B[请求到达Django中间件]
    B --> C{AuthenticationMiddleware}
    C --> D[JWTAuthentication 接管]
    
    D --> E[从Header提取Authorization字段]
    E --> F[获取JWT令牌字符串]
    
    F --> G[使用SECRET_KEY验证JWT签名]
    G --> H{令牌是否有效?}
    
    H -->|无效| I[返回认证错误响应]
    H -->|有效| J[从令牌解析user_id]
    
    J --> K[根据user_id查询数据库]
    K --> L[获取完整User模型对象]
    
    L --> M[将User对象附加到request.user]
    M --> N[进入Viewset处理业务逻辑]
    N --> O[返回API响应]

    style A fill:#e1f5fe
    style O fill:#e8f5e8
    style I fill:#ffebee
```

[[travel_of_an_api_request_in_django.png]](API 在  Django 中的旅程)

### `get_queryset()`覆写：实现数据权限控制

* **默认行为**：所有动作都基于开发者在 `ViewSet`中定义的静态的 `queryset`属性，例如`queryset = Drama.object.all()`。这会导致所有用户都能看到所有的 `Drama`对象，违背了我们用户只能看到自己建立的 `Drama` 对象的初衷。
* **我们的需求**：实现数据访问权限控制。
* **覆盖方式**：覆盖了 `.get_queryset()`方法。这个方法是所有需要查询数据的动作（比如 `.list()`、`.retrieve()`）的“数据源”。通过在这个方法内部根据 `self.request.user`动态过滤查询集，从源头上保证了用户只能访问到授权给他们的数据。

```Python
from rest_framwork import viewsets
from rest_framework.permissions import IsAuthenticated
from .serializer import DramaSerializer
from .models import Drama

class DramaViewSet(viewsets.ReadOnlyModelViewSet):
	serializer_class = DramaSerializer
	permission_classes = [IsAuthenticated]
	
	def get_queryset(self):
		return Drama.objects.filter(user=self.request.user)
```

>[!Tip] 注意：`ReadOnlyModelViewSet` 是专门为只读设计的 ViewSet。它继承自 `GenericViewSet`，混合了 `ListModelMixin`和 `RetrieveModelMixin`，提供了 `.list()`，对应 `GET /resource/`；以及 `.retrieve()`，对应  `GET /resource/{pk}`，用于获取单个资源详情。

> [!Tip] 覆盖方法（比如 `get_queryset()`和 `perform_create()`是 `ViewSet`内部预设流程中的“钩子”或“辅助方法”。它们的调用者是 `ViewSet` 的其他主方法（如 `.list()`、`create()`。这些主方法被调用时，`request`对象已经被作为参数传递给了它们，并且被存储为 `self.request`，因此，在这些辅助方法中，只要通过 `self.request`就可以访问到 `request`对象。 
### `@action`
 
前面写的覆写，是对 CRUD 行为进行修改和增强。当想要添加要给全新的、非标准的 API 接口时候（比如批量创建，查询 `SourceAudio`中所有出现过的不重复的 `season` 字段的值，就需要使用 `@action`装饰器。
#### `@action`接收的参数

* `detail`(必需参数)：决定了自定义动作作用于资源实例还是集合
	* `detail=True`表示这个动作是针对单个资源对象
		* `url`路径会包含该资源的主键，比如 `/resource/{pk}/my_action`；方法签名会接收 `pk` 参数，例如 `def my_action(self, pk=None)`
	* `detail=False`表示这个动作是针对集合
* `method`(必需参数)，比如 `methods=['get']`或 `methods=['post', 'get']`
* `url_path`（可选参数）：一个字符串，用于定义该动作在 URL 中的路径段。如果不提供，会根据方法名自动生成（例如，方法名为 `my_custom_action`，则`url_path`默认为`my-custom-action`。
* `url_name`（可选参数）：一个字符串，用于定义该动作的 URL 模式名称。主要用于反向解析 URL（例如 `reverse('my_action_name)`）
* `serializer_class`（可选参数）：允许为特定的 `action` 指定一个不同的序列化器，覆盖 `ViewSet`默认的 `serializer_class`。
* `permission_classes`（可选参数）：指定特殊的权限类，覆盖 `ViewSet`默认的`permission_clases`，比如有些动作可能需要更宽泛的权限
* `authentication_classes`（可选参数），类似上述两个。
#### 编写一个自定义 GET 接口

```Python
from rest_framework import viewset, parsers
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated
from .serializers import SourceAudioSerializer
from .models import SourceAudio

class SourceAudioViewSet(viewsets.ModelViewSet):
	serializer_class = SourceAudioSerializer
	permission_class = [IsAuthenticated]
	parser_classes = [parsers.MultiPartParser, parsers.FormParser]
	
	def get_queryset(self):
		return SourceAudio.objects.filter(user=self.request.user)
	# ...
	# ⬇️ 用 @action 装饰器添加批量创建 API 接口
	@action(detail=False, methods=['get'])
	def episodes(self, request):
		drama_id = request.query_params.get('drama_id')
		season = request.query_params.get('season')
		if not drama_id or not season:
			return Response({ "error": "drama_id and season parameters are required." })
		episodes = SourceAudio.objects.filter(
			drama_id=drama_id,
			season=season
		).values_list('episode', flat=True).distinct().order_by('episode')
		return Response(list(episodes))
		
```

> [!Tip] `@action`装饰器的方法是一个全新、独立的 API 入口，相当于在 ViewSet 内部定义了一个小型的、自成一体的“视图函数“。调用者是DRF的路由和分发系统。当一个请求的URL匹配到这个 `@action`时，DRF会把请求直接分发给这个方法来处理。因此，需要独立接收和处理来访的 `request`。
#### 编写一个批量创建数据接口

1. 使用 `@action`装饰器创建一个新的 API 接口。
2. 首先，需要自定义序列化器处理，让它可以接收一个列表。`get_serializer()`方法是 DRF 提供的一个辅助方法，它会获取 ViewSet 中定义的 `serializer_class`，并创建一个它的实例。
	- `data=request.data`：这将前端发送的请求体（JSON 数组）传递给序列化器，用于后续的验证和反序列化。
	- `many=True`：“魔法”所在。这个参数告诉序列器”我期望接收的是一个对象列表，而不是单个对象“。当 `many=True`被设置时，序列化器就会准备好去处理一个数组，并对数组中的每一个对象进行验证。
3. 之后会通过`serializer.is_valid(raise_exception=True)`进行验证。由于之前在 `get_serializer`中传入了 `many=True`，因此 `is_valid`会遍历 `request.data`数组中的每一个对象，并用 `serializer_class`中的序列化器去验证它们。如果数组中任何一个对象不符合规则（比如缺少字段、类型错误），`is_valid`就会抛出异常，DRF 会自动返回一个 400 Bad Request 响应，并附带详细的错误信息。
4. 如果所有的对象都验证通过，`serializer.validated_data`会成为一个包含多个 Python 字典的列表。对它进行 `for` 循环就可以遍历这个经过验证的、干净的数据列表，逐一处理。

```Python
from rest_framework import viewsets
from rest_framework.permissions import IsAuthenticated
from .serializers import AudioSliceSerializer

class AudioSliceViewSet(viewsets.ModelViewSet):
	serializer_class = 	AudioSliceSerializer
	permission_classes = [IsAuthenticated]
	
	def get_queryset(self):
		return AudioSlice.objects.filter(audio_chunk__source_audio__user=self.request.user)
		
	@action(detail=False, methods=['post'], url_path="create_batch")
	def create_batch(self, request):
		# 自定义序列化器，让它可以接收一个列表
		serializer = self.get_serializer(data=request.data, many=True)
		# 这行代码会触发验证
		serializer.is_valid(raise_exception=True)
		
		created_slices = []
		errors = []
		
		for item_data in serializer.validated_data:
			# required 数据，可以安全访问，保证不会抛出错误
			audio_chunk = item_data['audio_chunk']
			start_time = item_data['start_time']
			end_time = item_data['end_time']
			# 不确定字段是否存在于 item_data，需要使用安全的访问方式，避免抛出错误
			original_text = item_data.get('original_text', '')
			notes = item_data.get('notes', '')
			tags = item_data.get('tags', [])
			
			try:
				audio_slice_instance = slice_chunk_to_slice(audio_chunk, start_time, end_time)
				audio_slice_instance.original_text = original_text
				audio_slice_instance.notes = notes
				# 对于多对多关系，必须先保存主对象
				audio_slice_instance.save()
				# 然后才能添加或设置关联的对象
				audio_slice_instance.tags.set(tags)
				created_slices.append(audio_slice_instance)
			except Exception as e:
				errors.append(f"Error creating slice for chunk...")
		if errors:
			return Response({"message": "Some slices...", "errors":errors...})
		return Response(AudioSliceSerializer(created_slices, many=True).data, status=status...)	
```

#### 知识点
* 这里查找 `SourceAudio` 外键的 id，用的是 `drama_id`，因为在数据库层，外键的字段名的命名方式是`<外键字段名>_id`。Django ORM 提供了两种主要的方式来通过外键的 ID 进行过滤：
	* 直接使用实际的列名，也就是 `drama_id`，这简单而高效。
	* 使用双下划线 `__`遍历关系：`SourceAudio.objects.filter(drama__id=some_id_value).`
* `.values_list()`和 `.distinct()`都是 Django ORM 提供的、用于操作数据库查询结果集 (QuerySet) 的强大方法。它们通常被链接在 `.filter()`或 `.all()`之后，用于进一步“精炼”想要从数据库获取的数据，并且极大地优化性能：
	* `.values_list(*fields, flat=False)`
		* 作用：告诉 Django 不需要完整的模型对象，只需要每个对象一个或多个特定字段的值，相当于：`SELECT episode FROM ...`，这样大大减少了数据库负载和内存消耗
		* 返回结果：默认情况下（`flat=False`），返回一个元组列表（ `<QuerySet [(7,), (8,), (8,)]>`）。当使用`flat=True`，从一个”元组的列表“变成一个“值的列表”（`<QuerySet [7, 8, 8]`）。
	* `.disctinct()`：去重

#### 让 API 自解释：通过 URL 表达资源层级

我们之前将 `episodes`的读取 API 通过 `@action`装饰器放入了 `SourceAudioViewSet`中。
* URL：这样，就会产生这样一个 url：`/api/v1/audios/episodes/?drama_id=...&season=...`
* 语义上：这在语义上是成立的，也就是在 audios 集合中，为我找到指定 Drama ID 和 Season 的 Episodes 列表。

如果要以同样的思路写一个 `seasons API` 读取接口，会怎么样呢：
- URL： `/api/v1/audios/seasons/?drama_id=...`

如果放到 `DramaViewSet` 中：
- URL：`/api/v1/dramas/{id}/seasons`
显然这样更符合语义、符合用户的直觉：为我找到某个特定剧集的 Seasons 列表。

```Python
from rest_framework import viewsets
from rest_framework.response import Response
from .models import SourceAudio, Drama

class DramaViewSet(viewsets.ReadOnlyModelViewSet):
	# ...
	@action(detail=True, methods=['get'])
	def seasons(self, request, pk=None):
		drama = self.get_object()
		seasons = SourceAudio.objects.filter(drama=drama).values_list('season', flat=True).distinct().order_by('season') # ⬅️ 注意drama=drama
		return Response(list(seasons))
```

#### `ForeignKey`查询的幕后：对象与主键的自动转换

当向 `filter` 传入一个完整的模型对象来过滤一个 `ForeignKey`字段时，Django ORM 会自动使用该对象的主键来进行数据库查询。这既提供了代码层面的便利性（可以直接使用对象），又保证了数据库层面的高性能（只比较整数 ID）。这是 ORM 的一个核心优势。

```Python
seasons = SourceAudio.objects.filter(drama=drama)
```

#### 什么反向解析（Reverse URL Resolution）

通过 URL 的“名字”，动态地生成完整的 URL 路径字符串。

当 URL 结构发生变化时，比如从 `/users` 变成 `/profiles`，硬编码就会逼迫开发者必须逐一替换。反向解析就是为了解决这个问题。实现方式如下：

- 给 URL 一个名字
```python
# urls.py
from django.urls import path
from .views import PhraseLookupAPIView

urlpatterns = [
	path('lookup/', views.PhraseLookupAPIView.as_view(), name="phrase_lookup") # ← name这里
]
```
- 通过名字生成 URL。

#### `authentication_classes`和 `permission_classes`的区别
* `authentication_classes`：确认用户身份的合法性
* `permission_classes`：查看 `request.user`对象、根据预设规则判断该用户是否有权限执行当前操作。
