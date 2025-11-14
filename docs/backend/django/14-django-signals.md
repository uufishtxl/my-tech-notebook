# Django > 在钩子上挂载方法
## Django Signals

**Django Signals (`post_save`)**: 实现事件驱动的逻辑解耦。

我们可以利用这个特性，实现这样的目标场景：当一个 `SourceAudio` 模型被保存后，自动触发一个信号，通知 `slice_source_to_chunks` 服务开始执行“预切片”任务。

* `SourceAudio`模型在保存模型数据时，会发出信号。因此，只要监听这个信号，就能在 `SourceAudio`发出 `post_save`广播时，立即执行响应的操作。
### 在 `post_save()` hook上挂载任务

`post_save`信号：是 Django 内置的一个信号，会在任何模型 `.save()`方法成功执行之后被发送。

1. 利用 `@receiver`装饰器，将紧随其后的函数“注册”为一个信号接收器。
	- `sender`：发送信号的数据模型类
	- `instance`： 数据模型实例
	- `created`：一个布尔值，如果这是一次新建操作（SQL `INSERT`），`created`就为 `True`，如果这是一次更新操作（SQL `UPDATE`），`created` 就为 `False`。
	- `**kwargs`：一个字典，用来接收其他可能的参数。

```Python
# audio_slicer/signals.py
from django.db.models.signals import post_saven # 导入信号
from django.dispatch import receiver # 导入接收器
from .models import SourceAudio
from .services import slice_source_to_chunks

@receiver(post_save, sender=SourceAudio)
def source_audio_post_save(sender, instance, created, **kwargs):
	if created:
		slice_source_to_chunks(instance)
```
2. 定义好信号接收器后，还需要通知 Django 加载这个文件。Django 不会自动扫描项目中的 `signals.py`文件。
```Python
# audio_slicer/apps.py
from django.apps import AppConfig

class AudioSlicerConfig(AppConfig):
	# ...
	def ready(self):
		import audio_slicers.signals
```
