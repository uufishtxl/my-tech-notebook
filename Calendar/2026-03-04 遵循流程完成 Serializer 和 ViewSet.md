
## ReviewCard

该功能用于向用户展示按 SSR 间隔学习机制需要立即复习的 AudioSlice 卡片。

### `GET` `/api/v1/reviews/due/`

需要提供给前端的 JSON 数据格式：
```JSON
[
	{
		"id": 111, // 自带
		"box_level": 1, // 自带
		"next_review_date": "2026-03-05", // 自带
		"review_type": "translation",
		"slice_text": "Ah, that does it too.",
		"slice_translation": "啊，那也可以。",
		"audio_url": "http://lh:8000/media/chunks/Friends_S10_E12/chunk000.mp3",
		"start_time": 3.7,
		"end_time": 5.8,
		"audio_slice": 187
	}
]
```

### `POST` `/v1/reviews/${cardId}/submit/`
- 前端提供：`cardId: number` 和 `success: boolean`
- 更新后得到的状态：
```JSON
{
	"status": "",
	"new_level": 2,
	"next_review": "2026-3-11"
}
```

只需要对进来的数据进行验证，那这里需要指定一个专门的 Serializer吗？

### `PATCH` `/v1/audioslices/${sliceId}/`

其他问题：

- [ ] 建立 ReviewCard 数据是通过 signal 做的。但是 signal 不是不鼓励的吗？

## `AudioSlice`

### `POST` `/v1/audioslices/create_batch/`

目的：批量创建和更新 Audio Slices。
问题：
- [ ] 批量更新也用 POST 吗？这个不能是幂等？

前端期望获取的数据格式：
```JSON
[
	{
		"id": 144, // AudioSlice 本身的 pk → 自带
		"audio_chunk": 30, // audio_chunk 的 pk 默认就是给 id
		"start_time": 3.5, 
		"end_time": 4.9,
		"original_text": "Ah, that does it, too!", // 自带
		"highlights": [], // 自带,
		"is_pronounciation_hard": False, 
		"is_idiom": False,
		"created_at": "2026-02-20",
		"updated_at": "2026-03-01"
	}
]
```

### `GET` `/v1/audioslices/`

目的：前往需要获取某个Chunk所有的 AudioSlice Instances。

前端提供：`chunkId`

前往期望获取的数据格式：
```JSON
[
	{
		"id": 144, // AudioSlice 本身的 pk → 自带
		"audio_chunk": 30, // audio_chunk 的 pk 默认就是给 id
		"start_time": 3.5, 
		"end_time": 4.9,
		"original_text": "Ah, that does it, too!", // 自带
		"highlights": [], // 自带,
		"is_pronounciation_hard": False, 
		"is_idiom": False,
		"created_at": "2026-02-20",
		"updated_at": "2026-03-01"
	}
]
```

### `DELETE` `/v1/audioslices/${sliceId}/`

前端提供的：`sliceId`

前端期望返回的：`void`

## `AudioChunk`

### `GET` `/v1/audiochunks/

```JSON

```

### `GET` `/v1/audiochunks/${chunkId}/`

```JSON
{
	"id": 33,
	"source_audio":2,
	"chunk_index": 33,
	"file": "http://....",
	"has_slices":True,
	"title": "S10E13",
	"drama": "Friends",
}
```

### `POST` `/v1/audiochunks/${chunkId}/complete/`

目的：标记复习完成

前端提供的：`chunkId`

前端期望返回的：
```JSON
{
	"success": True,
	"current_chunk_id": 30,
	"current_index": 839, // 是啥？应该和序列化器没有关系
	"next_chunk_id": 31, // 或者 null
	"is_last": False,
	"total_chunks": 101, // 哪个total？本集 total？
	"message": "", // 什么 message
}
```
推断：应该也是和序列化器没关系。

## `SourceAudio`

### `GET` `/v1/audios/episodes/`

- [x] 完成

### `GET` `/v1/audios/lookup/`
- [x] 完成

### `POST` `/v1/audios/`

### `POST` `/v1/audios/${sourceAudioId}/upload_cover/`

- [x] 完成

知道一个 sourceaudio 的所有 chunks



### 🗺️ 全栈参数流转三维对照表

在 RESTful API 的世界里，向后端传非敏感参数，永远只有这两套标准姿势。请严格比对它们在三界的形态：

|**维度**|**🎯 姿势一：精准定位 (Path Variable)**|**🔍 姿势二：条件过滤 (Query Params)**|
|---|---|---|
|**💡 核心语境**|“我要找具体的那个它（比如 1号美剧）”|“把符合条件的这一批全给我挑出来”|
|**🌐 物理 HTTP 协议**<br><br>  <br><br>_(网线上跑的真实样子)_|`GET /api/v1/dramas/1/`|`GET /api/v1/episodes/?drama_id=1`|
|**💻 前端 Vue/Axios**<br><br>  <br><br>_(你怎么把数据发出去)_|**手工拼装字符串（必须自己加斜杠）**<br><br>  <br><br>``axios.get(`/api/v1/dramas/${id}/`)``<br><br>  <br><br>_(注：绝对不要写在 params 里)_|**利用 params 对象自动生成问号**<br><br>  <br><br>`axios.get('/api/v1/episodes/', {`<br><br>  <br><br>`params: { drama_id: 1 }`<br><br>  <br><br>`})`|
|**⚙️ 后端 Django/DRF**<br><br>  <br><br>_(你怎么把数据接过来)_|**躺着接（框架通过 URL 正则自动塞给你）**<br><br>  <br><br>`def seasons(self, request, pk=None):`<br><br>  <br><br>`print(pk) # 直接拿到 '1'`|**站着捞（自己去请求头里翻）**<br><br>  <br><br>`def list(self, request):`<br><br>  <br><br>`id = request.query_params.get('drama_id')`|
|**🧭 DRF 路由暗示**|`@action(detail=True)`<br><br>  <br><br>_(针对单个具体实例)_|`@action(detail=False)`<br><br>  <br><br>_(针对整个集合大海捞针)_|

---

### 📌 架构师的防坑口诀（背诵版）

下次你在写接口时，只要在脑子里默念这三句话，绝对不会再搞混：

1. **认清意图**：是**找人**（用斜杠，给 `pk`）还是**筛人**（用问号，塞 `params`）？
    
2. **前端不背锅**：Axios 的 `params` 对象**只负责生成问号和 `&` 符号**。如果你需要拼斜杠 `/1/`，Axios 不会帮你，你必须在 JavaScript 字符串里自己写死。
    
3. **后端不客气**：只要参数是在问号后面，不管是 1 个还是 10 个，后端统统去 `request.query_params` 这个字典里捞；只要参数是被两个斜杠夹住的，DRF 都会把它变成函数的命名参数（比如 `pk`）。