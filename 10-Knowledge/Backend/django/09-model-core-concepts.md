# Django > Model 核心知识点

## 外键

用途：用于建立模型间的关联。

当一个 Model 中使用类似如下的语句关联了外键时：

```Python
from django.db import models
class AudioChunk(models.Model):
	source_audio = models.ForeignKey(SourceAudio, on_delete=models.CASCADE)
```

* **数据库层**：Django 会在`AudioChunk`表中，创建一个名为 `source_audio_id`的列，这一列存储的就是它所关联的 `SourceAudio`记录的主键（ID）。这是关系数据库的标准工作方式，通过存储 ID 来建立表与表之间的关联，既高效又节约空间。
	这里逐一下这个数据库层中，外键列名的组成方式为 `<外键字段名'>_id`
* **Django ORM 层**：呈现的是对象。ORM 是一个位于Python代码和数据库之间的翻译官。因此在Python代码中访问 `mychunk.source_audio`时，ORM 会自动完成“幕后操作”，从而实现了链式调用：`my_chunk.source_auido.drama.name`。
* **Django 的 Admin 层**：为了用户友好，在访问 `AudioChunk`时，不会只显示一个光秃秃的数字，而是会获取到所有 `SourceAudio`对象，并使用在 `SourceAudio`中定义的`__str__`方法来生成一个可读、可供选择的列表。
## 联合键唯一约束

```Python
from django.db import models

class SourceAudio(models.Model):
# ...

class Meta:
	unique_together = ('drama',"season", "episode")

#...

```


当 Django 为这个模型生成数据表时，它会在 SQL 层面创建如下的 `UNIQUE`约束（以SQLite为例，其他数据库语法类似）：

```sql
CREATE TABLE audio_slicer_sourceaudio (
	id INTEGER PRIMARY KEY AUTOINCREMENT,
	drama_id INTEGER NOT NULL,
	season INTEGER NOT NULL,
	-- 外键约束
	FOREIGN KEY (drama_id) REFERRNCES audio_slicer_drama (id),
	UNIQUE(drama_id, season, episode)
)
```

## `FileField`

可以将 `FileField`理解为一个“文件管家”。将它附加在模型实例上，专门负责管理与这个实例关联的磁盘文件。

它有两个核心职责：

* 存储管理（写入文件）：
	* 当给它一个新文件时，它的工作是决定把这个文件存到哪里，并执行存储动作
	* “存到哪里”的规则就是 `upload_to`参数。
	* 标准的“给它一个新文件”的方式是传递一个 `File` 对象。这是最文件的方式，把文件交给“管家”，让它全权处理。
* 路径/URL 管理（读取文件信息）：
	* 当需要访问这个文件时，它的工作就是提供各种路径信息。
	* `.name`：数据库里记录的相对路径
	* `.path`：在服务器磁盘上的绝对路径
	* `.url`：在网络上的 URL 地址

让我们来实践一下，如何使用更稳健的传入 `File` 对象到 `FileField` 的方式来创建一个拥有`FileField` 类型字段的模型实例的流程。就以使用 `ffmpeg` 等分切割文件为例。

1. 用 `with`语句创建一个唯一的临时的磁盘目录，并通过 `subprocess` 运行 `ffmpeg`命令将一个 Source Audio 按 60 秒/分段进行切割，切割后的文件都会保存在这个临时目录上。
2. 用 `pathlib` 找到临时目录下所有的切割好的 chunk 文件，并对它们进行排序。
	- `Path(temp_dir)`创建一个 Path 对象，代表了 `temp_dir`这个临时目录的路径，包装为对象后，拥有各种文件系统操作方法。
	- 在这个路径对象中执行 `glob`方法（类似 `ls chunk_*.mp3`），通过模式匹配，找到所有符合`chunk_*.mp3`这个模式的文件，并逐个生成 `Path` 对象，注意，是`Path` 对象，不是文件。
	- `sorted(...)`：Python内置函数，接收一个可迭代的对象（例如列表、元组、迭代器），并返回一个新的列表，包含所有元素并按升序排列。`glob()`返回的正是一个迭代器，通过 `sorted`，就可以将这些文件按照文件名升序排列并返回一个列表，从而可以能够和`chunk_index` 一一对应。
3. 遍历已完成排序的 chunk 的 `Path` 对象，通过 `with` 语句安全的打开 chunk 文件，创建 File 对象实例，从而最终创建 `AudioChunk` 实例。

```Python
import uuid
import subprocess
import tempfile
import os
from pathlib import Path
from django.conf import settings
from django.core.files import File
from .models import SourceAudio, AudioChunk

def slice_audio_to_chunks(source_audio: SourceAudio):
	# 1.1 ffmpeg 需要指定文件输入地址
	source_file_path = source_audio.file.path
	
	"""
	1.2 使用 with 上下文管理器安全地创建唯一的临时磁盘目录
	确保在 with 代码块结束后总是清除该临时目录
	"""
	with tempfile.TemporaryDirectory() as temp_dir:
		# 1.3 ffmpeg 需要目标目录
		output_pattern = os.path.join(temp_dir, 'chunk_%03d.mp3') # %03d 是占位符
		try:
			# 1.4 组织 ffmpeg segment 命令
			command = [
				'ffmpeg',
				'-i', source_file_path,
				'-f', 'segment',
				'-segment_time', '60',
				'-c', 'copy',
				output_pattern				
			]
			subprocess.run(command, check=True, capture_output=True, text=True)
		except FileNotFoundError:
			raise Exception("ffmpeg is not installed or not in the system's PATH.")
		except subprocess.CalledProcessError as e:
			raise Exception(f'ffmpeg processing failed: {e.stderr}')
		
		"""
		"""
		chunk_files = sorted(Path(temp_dir).glob('chunk_*.mp3'))
		for i, chunk_file in enumerate(chunk_files):
			with open(chunk_file, 'rb') as f:
				AudioChunk.objects.create(
					source_audio=source_audio,
					chunk_index=i + 1,
					file=File(f, name=chunk_file.name)
				) 
```
