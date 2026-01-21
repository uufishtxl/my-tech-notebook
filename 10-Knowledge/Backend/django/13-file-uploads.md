# Django > 文件上传核心流程

知识点：
- **文件上传核心流程**: `models.FileField` 负责定义存储，`ViewSet` 中的 `parser_classes = [MultiPartParser]` 负责解析 `multipart/form-data` 请求，`ModelViewSet` 负责处理整个创建逻辑。

## 1. 创建 DB Table 字段

如果希望 Django 能够管理文件的上传和存储，并且在数据库中保存文件的相关信息（主要是路径），那么 `models.FileField`（或其子类 `ImageField`）就是标准且推荐的类。

核心作用和工作方式：

* 不会直接存储文件内容：`FileField`不会将文件的二进制内容直接存储在数据库中。这样效率低，并且会使数据库变得非常庞大
* 存储文件引用（路径）：这个字符串是文件相对 `settings.py`中 `MEDIA_ROOT`的相对路径。
* 管理文件存储：`FileField`会将实际的文件内容保存到服务器的文件系统上，具体位置由 `upload_to`参数决定。

关于 `upload_to`参数

目的：告诉 Django 文件应该上传到 `MEDIA_ROOT` 下的哪个子目录。可以是：

* 字符串，比如：`upload_to='audio_slicer/originals'`，所以文件都会上传到 `MEDIA_ROOT/audio_slicer/originals` 目录下。
* 可调用对象（函数）：这个函数会接收 `instance`（模型实例）和 `filename`（原始文件名）作为参数，然后返回一个动态生成的相对路径字符串。这使得文件组织更加灵活有条理。

```Python
from django.db import models

def audio_slice_upload_path(instance, filename):
	source = instance.audio_source
	drama_name = source.drama.name
	season_str = f"S{source.season:02d}"
	episode_str = f"E{source.episode:02d}"
	
	return f"audio_slicer/chunks/{drama_name}_{season_str}_{episode_str}/{filename}"
	

class AudioChunk(models.Model):
	# ...
	file = models.FileField(upload_to=audio_slice_upload_path)
```

访问文件信息：

当通过模型实例访问 `FileField`时，比如`source_audio_instance.file`，得到的是一个 `FileField` 对象，提供了很多遍历的属性和方法：

* `url`：返回公共 URL，用于在网页上访问
* `.path`：返回文件在服务器文件系统上的绝对路径
* `.name`：返回存储在数据库中的相对路径字符串
* `.size`返回文件大小
* `.read()` / `.open()` / `.close()`等：用于直接操作文件内容

## 2. API 请求（View 部分）

### 关于 `ModelViewSet`

可以继承 `viewsets.ModelViewSet`，或者子类 `viewsets.ReadOnlyModelViewSet`来编写 API 接口。如果需要对 CRUD 进行修改或增强，可以通过覆写特定方法，比如使用 `perform_create`修改和增强 POST 请求、覆写 `get_queryset`来做权限访问控制。

如果有较大的修改，则可以使用 `@action`装饰器进行定义。需要考虑到底应该写在哪个 `ViewSet` 下，从而保持 URL 的友好和语义高可读性。具体可以查看[这里](#ModelViewSet)。

### 文件上传解析器

要处理文件上传，会建议使用 `multipart/form-data`这个 `Content-Type`。因此必须在 View 中引入解析器进行解析。

```Python
from rest_framework import viewsets, parsers

class SomeViewSet(viewsets.ModelViewSet):
	#...定义 serializer_class / permission_clases之类的
	parser_classes = [parsers.MultiPartParser, parsers.FormParser]
	
```

### 处理批量上传

`ModelViewSet`的设计是一次处理一个实例。因此默认不支持批量上传。如果要支持批量上传，需要：
* 使用 `@action` 装饰器创建一个新的接口
* 自定义序列化器处理：需要能够接收一个文件列表和对应的元数据列表
* 复杂的文件解析：处理 multipart/form-data 中多个文件和对应元数据的映射也会比较复杂
